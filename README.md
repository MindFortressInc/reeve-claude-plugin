# Reeve plugin for Claude Code

A Claude Code plugin marketplace with one plugin, `reeve`. The plugin does three things:

1. It connects Claude to the remote Reeve MCP server at `https://api.meetreeve.com/mcp/directory/code`. That URL serves the Reeve tools the Claude directory lists: the same surface as the directory connector at `https://api.meetreeve.com/mcp/directory`, which leaves out the catch-all `use` tool, the `chat` agent, and tools that move money, generate images, video or audio with AI, or place AI voice calls.
2. It adds workflow skills: QuickBooks reports, CRM contacts, e-signature status, ad performance and memory search.
3. It turns on **automatic memory** in Claude Code and Cowork. Before each prompt, relevant memory from your Reeve account is recalled into context. Each turn is captured to your personal Reeve memory. Chat doesn't run plugin hooks, so there the plugin is the connector plus the skills.

[`plugins/reeve/README.md`](plugins/reeve/README.md) is the plugin's own description: exactly what the hooks send and where, how to turn capture off, and how to delete your data.

Nothing runs locally. The plugin is only configuration and skills: an MCP server entry, `mcp_tool` hooks that call that server over Claude Code's own OAuth, and `SKILL.md` files. It needs no API key, venv or local credential.

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

### Upgrading from an earlier version

Version 0.3.0 moved the plugin's server from `https://api.meetreeve.com/mcp/code` to `https://api.meetreeve.com/mcp/directory/code` (0.2.x used `/mcp/code`; 0.1.x used `/mcp`). The server name stays `plugin:reeve:reeve`, but Claude Code stores each MCP server's OAuth token under the server name plus a hash of its type, URL and headers, so the token saved for the old URL is not used for the new one. After updating, run `/mcp`, select `plugin:reeve:reeve` and sign in again. Until you do, the hooks fail as non-blocking errors, as described above.

The new URL serves the directory's tool surface, not the full one: `use`, `chat`, and the money, AI-generation and AI-voice tools are not on it. If you need those in Claude Code, add the full server under a different name with `claude mcp add --transport http reeve-full https://api.meetreeve.com/mcp` and keep the plugin for memory and skills.

### If you already have Reeve configured

- **claude.ai or Claude Desktop Reeve connector.** This can stay on. The plugin has its own URL, `https://api.meetreeve.com/mcp/directory/code`, which no connector uses. The connector URLs are `https://api.meetreeve.com/mcp`, `https://api.meetreeve.com/mcp/claude` and, for the connector listed in the Claude directory, `https://api.meetreeve.com/mcp/directory`. Claude Code suppresses a plugin server whose URL matches a claude.ai connector (debug log: `Lazy dedup: suppressing 1 plugin server(s) that duplicate claude.ai connectors: plugin:reeve:reeve`), and the memory hooks then have no server to call. Because the plugin's URL differs from every connector URL, the plugin and a connector coexist. The cost is that Claude Code shows Reeve's tools twice, once from each.
- **A manually added server at the same URL** (`https://api.meetreeve.com/mcp/directory/code`, added with `claude mcp add`, `.mcp.json` or `~/.claude.json`). Local, project and user servers take precedence over plugin servers, so Claude Code would hide this plugin's server and the memory hooks would have no server to call. Remove the manual entry and use the plugin.
- **A manually added server at another Reeve URL** (for example `https://api.meetreeve.com/mcp` or `https://api.meetreeve.com/mcp/code`). It doesn't hide the plugin, but Claude Code then shows Reeve's tools twice. Remove it with `claude mcp remove <name>` unless you want the full tool surface alongside the plugin.

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
- **Personal scope only.** Captured turns go to your **personal** Reeve memory, under the account you signed in with in `/mcp`. They are not written to an organization or team scope.
- **Opt out:** in the Reeve app, open **Settings > Knowledge > Your memory** ([meetreeve.com/settings/knowledge](https://meetreeve.com/settings/knowledge)) and switch off **Auto-write memories** (`auto_write_enabled`). Reeve then stores nothing the hooks send. To stop the hooks entirely, disable the plugin (`/plugin`) or uninstall it.
- **Delete:** **Forget…** on the same page removes the memory groups you choose; captured sessions are listed under **Other**. To have everything deleted, email [privacy@mindfortress.com](mailto:privacy@mindfortress.com).
- The plugin contains no credentials. Authentication is the OAuth token Claude Code holds for the `reeve` server.

## Claude's native auto-memory stays on

This plugin does **not** set `CLAUDE_CODE_DISABLE_AUTO_MEMORY`. Claude Code's built-in auto-memory keeps running alongside Reeve memory. That is a deliberate choice: native auto-memory stays on until Reeve's capture and distillation are proven to replace it. (A plugin can't set that variable anyway.)

## Layout

```
.claude-plugin/marketplace.json        marketplace "reeve", one plugin entry
plugins/reeve/.claude-plugin/plugin.json
plugins/reeve/README.md                the listing description: what is sent, opt out, delete
plugins/reeve/.mcp.json                server "reeve" -> https://api.meetreeve.com/mcp/directory/code
plugins/reeve/hooks/hooks.json         UserPromptSubmit + Stop mcp_tool hooks
plugins/reeve/skills/<name>/SKILL.md   workflow skills
```

Validate with:

```
claude plugin validate --strict .
claude plugin validate --strict plugins/reeve
claude plugin validate --profile directory plugins/reeve
```
