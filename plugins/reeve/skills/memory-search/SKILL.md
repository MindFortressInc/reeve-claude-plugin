---
name: memory-search
description: "Searches the user's personal Reeve memory: facts, decisions, preferences and rules they saved in Reeve, including facts Reeve condensed from captured Claude Code sessions. Reads the full text of a memory by id. Use when someone asks what they decided about something, what Reeve remembers about a person, project or company, whether they wrote something down before, or to look up a saved rule."
---

# Search your Reeve memory

Reeve memory is personal: only the signed-in user can read it. A search
covers every organization and space the user belongs to, and each hit names
its organization and space.

## Tools

| Tool | What it does |
| :- | :- |
| `memory_search` | Searches memory for a `query`. `kind` is `"fact"` (the default) or `"rule"`; `top_k` sets how many hits come back |
| `memory_read` | Reads up to five memories in full by `item_ids` |

## Workflow

1. **Search one concern at a time.** Call `memory_search` with the subject
   itself as the `query`: a person, company, product or project name, such
   as "Acme renewal" or "pricing page". Leave out dates and filler words,
   which match unrelated memories. For a saved rule or preference, pass
   `kind: "rule"`.
2. **Rephrase before concluding.** No hits means this query found nothing,
   not that no memory exists. Try a synonym or the shorter name before
   telling the user nothing is stored.
3. **Read conflicts in full.** When a hit carries a `conflict` or
   `possible_conflict` field naming other ids, call `memory_read` with those
   ids and show the user both versions. A hit marked `[AUTHORITATIVE]`
   outranks unmarked ones.
4. **Answer with the source.** Quote or summarize the memory, and say which
   organization or space it came from.
