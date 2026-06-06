# 0001 — Depend on `@ankimcp/anki-mcp-server` rather than building our own

- Status: accepted
- Date: 2026-06-06

## Context

ankify needs to talk to Anki. The integration point is AnkiConnect, an Anki add-on exposing a local
HTTP/JSON-RPC API on `http://localhost:8765`. We can either (a) reference an existing MCP server that
wraps AnkiConnect, or (b) build our own thin MCP server (an HTTP client to `:8765`) exposing only the
handful of actions we use.

The plugin's real value is the **card-authoring methodology** (atomic cards, Basic-vs-Cloze choice,
dedup, deck targeting, the review gate) — not the AnkiConnect plumbing, which is a solved problem.

## Decision

Reference `@ankimcp/anki-mcp-server` via `npx` in `.mcp.json`:

```json
{ "command": "npx", "args": ["-y", "@ankimcp/anki-mcp-server", "--stdio"] }
```

It is MIT-licensed, npx-runnable (zero install), actively maintained, and exposes every action v1
needs — `addNote`/`addNotes`, `updateNoteFields`, `findNotes`/`notesInfo`, `deckNames`/`createDeck`,
`modelNames`/`modelFieldNames`, and `storeMediaFile`.

## Consequences

- **Upside:** no MCP server code to write, test, or maintain; battle-tested tool schemas; faster to
  ship.
- **Downside:** we inherit its broad (~40) tool surface, which is more for the model to wade through
  than a curated set, and we take on upstream-breakage risk (a release could change tool names that
  the `anki-cards` skill references).
- **Reversal path (cheap, isolated):** swap the `command`/`args` in `.mcp.json` to point at a thinner
  existing server or our own implementation. The skill references actions by their AnkiConnect names,
  which are stable, so the blast radius of a swap is one config file plus any tool-name aliasing.
