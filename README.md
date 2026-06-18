# ankify

A [Claude Code](https://code.claude.com) plugin that turns your material (a PDF, an image, or
pasted text) or the current conversation into [Anki](https://apps.ankiweb.net/) flashcards, and
edits existing notes. It drafts cards, shows them to you for review, then adds them to Anki via the
[AnkiConnect](https://git.sr.ht/~foosoft/anki-connect) add-on.

Claude reads the material and writes the card content (including Cloze deletions). The AnkiConnect
MCP server only stores media and creates or updates notes.

## What it can do

- **Cards from material.** Give it a PDF, image, or text and it drafts cards.
- **Cards from chat.** "Make a card out of what we just discussed."
- **Edit existing cards.** Find a note and update its fields or tags.
- **Curate existing cards.** Audit a deck, a tag, or your leeches against the same quality rubric it
  writes by. It shows a read-only triage (worst-first), then fixes on your approval — rewriting,
  splitting, retagging, or moving cards while preserving their scheduling history. Cards it can't
  salvage are tagged for your attention rather than deleted (deletes need your explicit per-card OK).
- It picks the right note type per card — Basic, Basic (and reversed), type-in-the-answer, Cloze, or
  Image Occlusion — places cloze blanks on the part you're actually studying, suggests a target deck,
  adds tags, and skips duplicates. The review step shows each card **the way Anki will test it**, with
  a one-line note on what it tests, before anything is written.
- It lays cards out to read cleanly — one item per line, with distinct groups (definitions, examples)
  spaced apart instead of run together on a single line — so they're easy to read on your phone.
  Curation can retrofit this onto existing cramped cards too.
- **Optional personal profile.** Drop a `~/.ankify/profile.md` (ankify offers to create one on first
  run) to tell it your subjects/languages, which scripts you can type, and how plain or styled you
  like your cards. It's machine-local and never leaves your computer; without it, ankify behaves
  generically.
- It auto-syncs to AnkiWeb after adding or editing cards, so changes reach your phone and other
  devices without a manual sync. This needs AnkiWeb set up in the Anki desktop app. If a sync can't
  go through, ankify reports it, but the card still saves locally.

## Prerequisites

1. **Anki desktop**, installed and **running** (the API only works while Anki is open, not with
   AnkiWeb, AnkiMobile, or AnkiDroid alone).
2. **AnkiConnect add-on**: in Anki, Tools → Add-ons → Get Add-ons → paste code **`2055492159`**, then
   restart Anki. It listens on `http://localhost:8765`.
3. **Node.js ≥ 22.12** on your PATH (the bundled MCP server runs via `npx`).

## Install

```text
/plugin marketplace add matthewalton/ankify
/plugin install ankify@ankify-marketplace
```

This also configures the `anki` MCP server automatically, so there's no manual MCP setup.

### Local testing (from a clone)

```text
/plugin marketplace add .
/plugin install ankify@ankify-marketplace
```

## Usage

With Anki open, just ask and the skill triggers automatically:

- "Ankify this PDF" (with a file attached). It drafts cards, shows them, and adds them on your OK.
- "Make an Anki card for the fact that the mitochondrion is the powerhouse of the cell."
- "Edit my Anki note about the French verb *être* and fix the past tense."

Or use the explicit commands:

- `/ankify:add [file or description]` creates cards from material or chat.
- `/ankify:edit [which note and what to change]` updates an existing note.
- `/ankify:curate [deck, tag, or leeches]` audits existing cards and fixes the bad ones (defaults to
  your leeches).

By default ankify shows drafts for review before writing to Anki. Say **"just add them"** to skip the
review for a batch you trust.

## Uninstall

```text
/plugin uninstall ankify@ankify-marketplace
```

## Roadmap

Candidates for later: study and review automation, collection stats, custom note-type creation (with
bespoke CSS), and full-collection discovery sweeps (curation today is scoped to a deck, a tag, or your
leeches).

## Contributing

Issues and PRs welcome. Keep the plugin's value in the card-authoring methodology
(`skills/anki-cards/SKILL.md`); the MCP server is an external dependency (see
[`docs/adr/0001`](docs/adr/0001-depend-on-ankimcp-server.md)).

Commits follow [Conventional Commits](https://www.conventionalcommits.org/). See
[CONTRIBUTING.md](CONTRIBUTING.md) for the format and types.

## License

[MIT](LICENSE) © Matt Alton.

The bundled AnkiConnect integration uses [`@ankimcp/anki-mcp-server`](https://github.com/ankimcp/anki-mcp-server).
