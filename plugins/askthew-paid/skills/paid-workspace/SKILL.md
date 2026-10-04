---
description: Paid Ask The W plugin for coding agents. Bind a project, pull context at session start, and push decisions or updates when the person asks.
---

# Ask The W Paid Workspace

Use this skill when working in a Claude Code repo with the paid Ask The W plugin enabled.

## What This Plugin Is

This is the Ask The W plugin for coding agents. It starts the MCP server with `ASKTHEW_REQUIRE_PAID=true`, so protected tools need a paid project bind before normal use.

It is one plugin. Bind is required. The paid project is the source of context and the place later writes go. Local work stays local until the person asks to push an update.

If a tool returns `free_limit_reached`, treat that as a server-side configuration problem for the paid install and tell the user.

## Connected Through The Hosted Address

If Ask The W is connected through `https://my.askthew.com/api/mcp` (Claude, ChatGPT, or another AI app), there is no bind step. Sign-in already chose a starting project.

- `list_projects` lists the projects the person can connect. `choose_project` switches to one, by `project_id` or by a working preview address. `create_project` returns the link that creates a new project in the web app.
- Ask W's own tools are here too, for example `way_board_read`, `create_decision`, `create_outcome`, `set_card_field`, `add_to_way`, and `files_answer`. Each one runs the same checks as the web app. Call a write only when the person asks.
- After `claim_work`, keep the `leaseToken` it returns. Pass it to `heartbeat_work`, `complete_work`, and `release_work`.

## Session Start

Call `get_project_context` at session start, or read the `askthew://project-context` resource. It returns one compact, cited view of the bound project: the current W Card, the North Star, active outcomes, open Way decisions, current moves, and current artifacts.

If the install is bound, also call `list_ready_work`. Present the ready work. Never claim it. Never send a signal.

Do not call `capture_session_signal` at session start. There is no `setup_complete` cadence. There are no checkpoints after every step.

## If The Install Is Unbound

If `get_project_context` reports that this install is not connected to a project:

1. Call `askthew_start_workspace_bind`.
2. Ask the user to open the returned URL and confirm the displayed code in Ask The W.
3. Call `askthew_check_workspace_bind({ codeDisplay })` until it reports `completed`.
4. Retry `get_project_context`. If bound, also call `list_ready_work`.

Bind is the only doorway. There are no signup tools in this plugin. Do not try to sign the user up from the terminal. Do not ask for an email or a six-digit code.

## Local Work Stays Local

Keep coding, tests, and files on the machine. Do not upload transcripts. Do not read JSONL transcript files. Do not call `capture_session_signal` unless the user explicitly asks to push an update.

## Push An Update Only When Asked

When the user explicitly asks to push an update:

1. Summarize local work since `~/.askthew/last-pull.json` from git (`git log` / `git diff` since `gitHead` or `pulledAt`) plus what this chat already knows.
2. If a run is claimed, call `complete_work` with a summary, files touched, checks, and commit. Include optional `stack` from repo markers when the stack is known.
3. If they asked to record a choice, call `create_decision`.
4. Else if the work is project-relevant, send one `capture_session_signal` (`implementation_update` or `final_summary`).
5. Advance the last-pull cursor after a successful write.

Do not call `capture_session_signal` for any other reason. Do not send transcripts, secrets, credentials, private tokens, or large copied content.

## Coding-Agent Work

`list_ready_work` returns Way board rows that a person has OKed for a coding agent. Claim one with `claim_work` only when the user chooses an item. After you claim, read the files on that ticket and the files `list_project_docs` returns. Always read `CLAUDE.md` and `AGENTS.md`. Read the files for the outcome you are publishing on the working preview. Do not wait for the person to ask. Then `heartbeat_work`, and finish with `complete_work` or `release_work`. On `complete_work`, report the stack from repo markers when it is known.

If it returns nothing, read the `emptyReason` it gives back rather than guessing: `no_eligible_moves`, `no_approved_requests`, or `no_queued_runs` each name a different missing step.

## Other Products

Call these only when the user asks, except the standing-doc read at claim:

- `list_project_docs` for Project files the working preview is built from. `read_project_doc` / `write_project_doc` for a named Workspace file.
- `draft_wire_message` drafts a Wire message. Draft only. It never sends.
- `request_world_analysis` and `request_whiteboard_map` request work from World and Whiteboard. Both return a run id to poll, and both go through the same review path as those product UIs.
