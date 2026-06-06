# ankify

A [Claude Code](https://code.claude.com) plugin that turns your material — a PDF, an image, or
pasted text — or the current conversation into well-formed [Anki](https://apps.ankiweb.net/)
flashcards, and edits existing notes. It drafts cards, shows them to you for review, then adds them
to Anki via the [AnkiConnect](https://git.sr.ht/~foosoft/anki-connect) add-on.

Claude reads and understands the material and writes the card content (including Cloze deletions);
the AnkiConnect MCP server only stores media and creates/updates notes.

## What it can do (v1)

- **Cards from material** — give it a PDF, image, or text and it drafts cards.
- **Cards from chat** — "make a card out of what we just discussed".
- **Edit existing cards** — find a note and update its fields or tags.
- Picks **Basic vs Cloze** per card, suggests a **target deck**, tags for findability, and skips
  duplicates — all shown in a **review step** before anything is written to Anki.
- **Auto-syncs to AnkiWeb** after adding or editing cards, so changes reach your phone and other
  devices without a manual sync. Needs AnkiWeb set up in the Anki desktop app; if a sync can't go
  through it's reported but never blocks the card from being saved locally.

## Prerequisites

1. **Anki desktop**, installed and **running** (the API only works while Anki is open — not
   AnkiWeb/AnkiMobile/AnkiDroid alone).
2. **AnkiConnect add-on**: in Anki, Tools → Add-ons → Get Add-ons → paste code **`2055492159`**, then
   restart Anki. It listens on `http://localhost:8765`.
3. **Node.js ≥ 22.12** on your PATH (the bundled MCP server runs via `npx`).

## Install

```text
/plugin marketplace add matthewalton/ankify
/plugin install ankify@ankify-marketplace
```

This also configures the `anki` MCP server automatically — no manual MCP setup needed.

### Local testing (from a clone)

```text
/plugin marketplace add .
/plugin install ankify@ankify-marketplace
```

## Usage

With Anki open, just ask — the skill triggers automatically:

- "Ankify this PDF" (with a file attached) — drafts cards, shows them, adds on your OK.
- "Make an Anki card for the fact that the mitochondrion is the powerhouse of the cell."
- "Edit my Anki note about the French verb *être* — fix the past tense."

Or use the explicit commands:

- `/ankify:add [file or description]` — create cards from material or chat.
- `/ankify:edit [which note and what to change]` — update an existing note.

By default ankify shows drafts for review before writing to Anki. Say **"just add them"** to skip the
review for a batch you trust.

## Uninstall

```text
/plugin uninstall ankify@ankify-marketplace
```

## Roadmap

Not in v1, but candidates for later: study/review automation, collection stats, custom note-type
creation, and bulk reorganization.

## Contributing

Issues and PRs welcome. Keep the plugin's value in the card-authoring methodology
(`skills/anki-cards/SKILL.md`); the MCP server is an external dependency (see
[`docs/adr/0001`](docs/adr/0001-depend-on-ankimcp-server.md)).

Commits follow [Conventional Commits](https://www.conventionalcommits.org/) — see
[CONTRIBUTING.md](CONTRIBUTING.md) for the format and types.

## License

[MIT](LICENSE) © Matt Alton.

The bundled AnkiConnect integration uses [`@ankimcp/anki-mcp-server`](https://github.com/ankimcp/anki-mcp-server).
