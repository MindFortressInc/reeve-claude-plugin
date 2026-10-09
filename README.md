# Reeve plugin for Claude Code

A Claude Code plugin marketplace with one plugin, `reeve`. The plugin does two things:

1. It connects Claude Code to the remote Reeve MCP server at `https://api.meetreeve.com/mcp/code`, which provides the full Reeve tool surface.
2. It turns on **automatic memory**. Before each prompt, relevant memory from your Reeve account is recalled into context. Each turn is captured to your personal Reeve corpus.

Nothing runs locally. The plugin is only configuration: an MCP server entry plus `mcp_tool` hooks that call that server over Claude Code's own OAuth. It needs no API key, venv or local credential.

## Install

In Claude Code:

```
/plugin marketplace add MindFortressInc/reeve-claude-plugin
/plugin install reeve@reeve
```

Then authenticate once:

```
/mcp
```

Select `plugin:reeve:reeve` and complete the browser sign-in with your Reeve account. The hooks never start a sign-in themselves. Until the server is authenticated and connected, each hook fails as a non-blocking error (`MCP server 'plugin:reeve:reeve' not connected`). Claude Code may show a `hook error` notice, and your session continues normally.

To remove it: `/plugin uninstall reeve@reeve`.

### Upgrading from 0.1.x

Version 0.2.0 moved the plugin's server from `https://api.meetreeve.com/mcp` to `https://api.meetreeve.com/mcp/code`. The server name stays `plugin:reeve:reeve`, but Claude Code stores each MCP server's OAuth token under the server name plus a hash of its type, URL and headers, so the token saved for the old URL is not used for the new one. After updating, run `/mcp`, select `plugin:reeve:reeve` and sign in again. Until you do, the hooks fail as non-blocking errors, as described above.

### If you already have Reeve configured

- **claude.ai or Claude Desktop Reeve connector.** This can stay on. The plugin has its own URL, `https://api.meetreeve.com/mcp/code`, which serves the same Reeve tools as the connector URLs (`https://api.meetreeve.com/mcp` and `https://api.meetreeve.com/mcp/claude`). Claude Code suppresses a plugin server whose URL matches a claude.ai connector (debug log: `Lazy dedup: suppressing 1 plugin server(s) that duplicate claude.ai connectors: plugin:reeve:reeve`), and the memory hooks then have no server to call. Because no connector uses `/mcp/code`, the plugin and a connector on `/mcp` or `/mcp/claude` coexist.
- **A manually added server at the same URL** (`https://api.meetreeve.com/mcp/code`, added with `claude mcp add`, `.mcp.json` or `~/.claude.json`). Local, project and user servers take precedence over plugin servers, so Claude Code would hide this plugin's server and the memory hooks would have no server to call. Remove the manual entry and use the plugin.

## What it captures

| Event | Tool called | Sent to Reeve |
| :- | :- | :- |
| `UserPromptSubmit` | `memory_capture_turn` (`role: "user"`) | The prompt you submitted |
| `Stop` | `memory_capture_turn` (`role: "assistant"`) | Claude's final message for that turn |

Each capture also carries the Claude Code `session_id` and the working directory (`cwd`), so turns group into one transcript per session.

It does **not** capture tool calls, tool output, file contents or Claude's intermediate messages. It captures only what you asked and the conclusion Claude reached.

## What it recalls

On every `UserPromptSubmit`, `memory_recall_context` receives your prompt, `session_id` and `cwd`. It returns memory relevant to that prompt, which Claude Code adds to the turn's context. If nothing is relevant, it returns nothing and no context is added.

Recall runs on `UserPromptSubmit`, not `SessionStart`: Claude Code skips `mcp_tool` hooks on `SessionStart` at launch, because MCP servers aren't connected yet.

Both `UserPromptSubmit` hooks have a 15-second timeout, below Claude Code's 30-second default for that event. The `Stop` capture also has a 15-second timeout.

## Privacy

- **What leaves your machine:** the full text of every prompt you submit (including anything you paste into it) and of Claude's final reply each turn, plus the session id and working-directory path. Don't paste secrets into prompts with the plugin enabled.
- **Personal scope only.** Captured turns go to your **personal** Reeve corpus, under the account you signed in with in `/mcp`. They are not written to an organization or team scope.
- **Opt out:** turn off automatic memory writes in your Reeve memory settings (`auto_write_enabled`). To stop the hooks entirely, disable the plugin (`/plugin`) or uninstall it.
- The plugin contains no credentials. Authentication is the OAuth token Claude Code holds for the `reeve` server.

## Claude's native auto-memory stays on

This plugin does **not** set `CLAUDE_CODE_DISABLE_AUTO_MEMORY`. Claude Code's built-in auto-memory keeps running alongside Reeve memory. That is a deliberate choice: native auto-memory stays on until Reeve's capture and distillation are proven to replace it. (A plugin can't set that variable anyway.)

## Layout

```
.claude-plugin/marketplace.json        marketplace "reeve", one plugin entry
plugins/reeve/.claude-plugin/plugin.json
plugins/reeve/.mcp.json                server "reeve" -> https://api.meetreeve.com/mcp/code
plugins/reeve/hooks/hooks.json         UserPromptSubmit + Stop mcp_tool hooks
```

Validate with:

```
claude plugin validate --strict .
claude plugin validate --strict plugins/reeve
```
