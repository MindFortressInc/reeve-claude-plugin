# Reeve

Reeve is a workspace for running a business's customers, money and marketing in one place: a CRM, QuickBooks reports, e-signatures, connected ad accounts, video, and a personal memory that carries what you've decided from one conversation to the next. This plugin connects Claude to your Reeve account so Claude can look things up and get work done there, and adds skills for the jobs people ask for most.

You need a Reeve account. Sign up at [meetreeve.com](https://meetreeve.com).

## What it adds

- **The Reeve connector** at `https://api.meetreeve.com/mcp/directory/code`. You sign in with your Reeve account; the plugin holds no key or password.
- **Skills** for common Reeve work:
  - `finance-reports`: a profit and loss, an accounts receivable aging or an accounts payable aging from your QuickBooks companies.
  - `crm-contacts`: find a contact, read their record and history, update their tags, stage or owner, and add a note.
  - `signature-status`: check who has signed an e-signature envelope, and preview its pages.
  - `ad-performance`: spend and results from your connected Meta, Google Ads and TikTok accounts.
  - `memory-search`: search your personal Reeve memory.
- **Automatic memory** in Claude Code and Cowork: recall before each prompt and capture after each turn, described below.

## Where each part runs

| | Connector and skills | Automatic memory (hooks) |
| :- | :- | :- |
| Claude Code | Yes | Yes |
| Cowork | Yes | Yes |
| Chat (claude.ai and the Claude apps) | Yes | No. Chat doesn't run plugin hooks, so nothing is captured or recalled automatically there |

## Automatic memory: what is sent and where

In Claude Code and Cowork the plugin runs three hooks. Each one calls a Reeve tool on `api.meetreeve.com` over the connection you signed in with.

| When | Reeve tool | What is sent |
| :- | :- | :- |
| You submit a prompt | `memory_capture_turn` | The prompt text, the session id and the working directory. Stored in your memory |
| You submit a prompt | `memory_recall_context` | The prompt text, the session id and the working directory. Used to look up relevant memory and not stored |
| Claude finishes its reply | `memory_capture_turn` | Claude's final reply for that turn, the session id and the working directory. Stored in your memory |

That is everything the hooks send. **Tool calls, tool output, file contents and Claude's intermediate messages are not sent.**

- **Where it goes:** your **personal** Reeve memory, under the account you signed in with. Only you can read it. It is not shared with your organization or team.
- **What Reeve does with it:** Reeve removes credentials, tokens and personal identifiers it recognizes before storing a turn. It indexes captured text for search, and may condense each day's captured turns into short facts in the same personal memory. Recall returns memory relevant to your prompt, which Claude adds to that turn's context; when nothing is relevant, nothing is added.
- The full text of each prompt leaves your machine, including anything you paste into it. Don't paste secrets into prompts while capture is on.

How Reeve handles your data, including the AI providers it shares data with, is in the [Reeve privacy policy](https://meetreeve.com/privacy).

## Turn capture off

- **Stop storing turns:** in the Reeve app, open **Settings > Knowledge > Your memory** ([meetreeve.com/settings/knowledge](https://meetreeve.com/settings/knowledge)) and switch off **Auto-write memories**. Reeve then stores nothing the hooks send. Recall keeps working from the memory you already have.
- **Stop the hooks entirely:** disable the plugin with `/plugin` in Claude Code, or uninstall it (`/plugin uninstall reeve@reeve`). Nothing is sent after that.

## Delete your data

- In **Settings > Knowledge > Your memory**, use **Forget…** to choose which groups of your memory to remove. Captured Claude Code sessions are in your personal memory there, under **Other**. You can also delete individual memories from the list on that page.
- To have everything Reeve stores about you deleted, email [privacy@mindfortress.com](mailto:privacy@mindfortress.com).

## Install

In Claude Code 2.1.275 or later:

```
/plugin install reeve --marketplace MindFortressInc/reeve-claude-plugin
```

On an older Claude Code, add the marketplace first:

```
/plugin marketplace add MindFortressInc/reeve-claude-plugin
/plugin install reeve@reeve
```

## Sign in

In Claude Code, run `/mcp`, select `plugin:reeve:reeve` and sign in with your Reeve account in the browser. Until you do, the memory hooks fail as non-blocking errors: Claude Code may show a `hook error` notice, and your session continues normally.

If you also have the Reeve connector on in claude.ai, it can stay on. The plugin uses its own URL so Claude Code keeps both, and you'll see Reeve's tools twice in Claude Code.

## Privacy and license

- Privacy policy: [meetreeve.com/privacy](https://meetreeve.com/privacy)
- License: MIT
- Source: [github.com/MindFortressInc/reeve-claude-plugin](https://github.com/MindFortressInc/reeve-claude-plugin)
