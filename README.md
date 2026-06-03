# Ask The W Claude Plugins

Marketplace for installing the paid Ask The W plugin in Claude Code.

## Team Install

Add this to a repository's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "askthew": {
      "source": { "source": "github", "repo": "AskTheWAI/claude-plugins" },
      "autoUpdate": true
    }
  },
  "enabledPlugins": {
    "askthew-paid@askthew": true
  }
}
```

Claude Code prompts each teammate to trust the marketplace and install the plugin when they open the repo.

## Personal Install

```text
/plugin marketplace add AskTheWAI/claude-plugins
/plugin install askthew-paid@askthew
```

For project scope from the terminal:

```bash
claude plugin marketplace add AskTheWAI/claude-plugins --scope project
claude plugin install askthew-paid@askthew --scope project
```

## First-Time Signup

The paid plugin uses first-run signup and workspace binding through MCP tools. If a protected tool returns `needs_signup`, ask for the user's email, call `askthew_start_signup({ email })`, ask for the six-digit email code, then call `askthew_complete_signup({ email, code })`. If a protected tool returns `needs_paid_workspace`, call `askthew_start_workspace_bind`, ask the user to confirm the returned code in Ask The W, then call `askthew_check_workspace_bind` until it reports `completed`.

After binding, the plugin is the paid/unlimited surface for signals, decisions, recaps, coaching, and next-move capture.
