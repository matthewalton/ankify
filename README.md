# ankify

A [Claude Code](https://code.claude.com) plugin that turns your material (a PDF, an image, or
pasted text) or the current conversation into [Anki](https://apps.ankiweb.net/) flashcards, and
edits existing notes. It drafts cards, shows them to you for review, then adds them to Anki via the
[AnkiConnect](https://git.sr.ht/~foosoft/anki-connect) add-on.

Claude reads the material and writes the card content (including Cloze deletions). The AnkiConnect
MCP server only stores media and creates or updates notes.

## What it can do (v1)

- **Cards from material.** Give it a PDF, image, or text and it drafts cards.
- **Cards from chat.** "Make a card out of what we just discussed."
- **Edit existing cards.** Find a note and update its fields or tags.
- It picks Basic or Cloze per card, suggests a target deck, adds tags for findability, and skips
  duplicates. All of this shows up in a review step before anything is written to Anki.
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

By default ankify shows drafts for review before writing to Anki. Say **"just add them"** to skip the
review for a batch you trust.

## Uninstall

```text
/plugin uninstall ankify@ankify-marketplace
```

## Roadmap

Not in v1, but candidates for later: study and review automation, collection stats, custom note-type
creation, and bulk reorganization.

## Contributing

Issues and PRs welcome. Keep the plugin's value in the card-authoring methodology
(`skills/anki-cards/SKILL.md`); the MCP server is an external dependency (see
[`docs/adr/0001`](docs/adr/0001-depend-on-ankimcp-server.md)).

Commits follow [Conventional Commits](https://www.conventionalcommits.org/). See
[CONTRIBUTING.md](CONTRIBUTING.md) for the format and types.

## License

[MIT](LICENSE) © Matt Alton.

The bundled AnkiConnect integration uses [`@ankimcp/anki-mcp-server`](https://github.com/ankimcp/anki-mcp-server).
