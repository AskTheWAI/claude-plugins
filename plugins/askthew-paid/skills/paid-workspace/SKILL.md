---
description: Paid Ask The W Claude Code workflow for workspace-bound project context, signals, decisions, next moves, recaps, coaching, and workspace binding.
---

# Ask The W Paid Workspace

Use this skill when working in a Claude Code repo with the paid Ask The W plugin enabled.

## What This Plugin Is

This is the paid Claude Code plugin wrapper. It is separate from `@askthew/mcp-plugin`, which remains the reusable MCP server package. The plugin starts that server with `ASKTHEW_REQUIRE_PAID=true`, so protected tools require a paid workspace binding before normal use.

Paid workspace-bound installs are intended for unlimited signals, decisions, recaps, coaching, and next-move capture. If a tool returns `free_limit_reached`, treat that as a server-side configuration problem for the paid install and tell the user.

## Read The Project First

Call `get_project_context` at session start, or read the `askthew://project-context` resource. It returns one compact, cited view of the bound project: the current W Card, the North Star, active outcomes, open Way decisions, current moves, and current artifacts.

Prefer it over `recap`. Recap summarizes only this install's own trail, not the project.

## Required Startup Flow

Call `capture_session_signal` with `kind: "setup_complete"` at session start. Omit `scopeKey` unless deliberately overriding the workspace default.

If the tool reports that this install is not connected to a project:

1. Call `askthew_start_workspace_bind`.
2. Ask the user to open the returned URL and confirm the displayed code in Ask The W.
3. Call `askthew_check_workspace_bind({ codeDisplay })` until it reports `completed`.
4. Retry the original `setup_complete` capture.

Bind is the only doorway. There are no signup tools in this plugin; do not try to sign the user up from the terminal.

## Capture Cadence

Send compact signals with `capture_session_signal`:

- `setup_complete` at session start.
- `session_checkpoint`, `direction_change`, or `implementation_update` after meaningful progress or state changes.
- `verification_result` after tests, builds, validation, or manual QA.
- `final_summary` before the final reply.

Keep payloads short: summary, files touched, commands run, and useful metadata. Do not send transcripts, secrets, credentials, private tokens, or large copied content.

`capture_session_signal` is the only tool you may call without the user explicitly asking. Every other tool needs an explicit request.

## Decisions

Use `create_decision` when a product, architecture, implementation, release, or workflow choice should be durable. Link source signals with `sourceSignalIds` when available.

## Coding-Agent Work

`list_ready_work` returns Way board rows that a person has OKed for a coding agent. Claim one with `claim_work`, then `heartbeat_work`, and finish with `complete_work` or `release_work`.

If it returns nothing, read the `emptyReason` it gives back rather than guessing: `no_eligible_moves`, `no_approved_requests`, or `no_queued_runs` each name a different missing step.

## Other Products

- `list_project_docs` / `read_project_doc` / `write_project_doc` for Workspace files.
- `draft_wire_message` drafts a Wire message. Draft only. It never sends.
- `request_world_analysis` and `request_whiteboard_map` request work from World and Whiteboard, only when the user explicitly asks. Both return a run id to poll, and both go through the same review path as those product UIs.

## Recovery

If startup capture was missed, send `setup_complete` as soon as you notice with `metadata.recovered_missed_startup=true`.
