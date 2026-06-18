# Context — ankify

The shared language of this project. This file is a **glossary only** — no implementation details,
no decisions (those live in `docs/adr/`), no roadmap.

## Glossary

- **Material** — source content the user provides (PDF, image, pasted text) that cards are derived
  from. ankify reads it; AnkiConnect never parses it.
- **Draft** — a card ankify has generated but not yet committed to Anki. A draft lives only in the
  chat review.
- **Review (gate)** — the step where drafts are shown to the user for approval / edit / cull before
  any note is created or updated in Anki.
- **Commit** — the act of pushing approved drafts into Anki (via `addNotes`) or applying an approved
  change (via `updateNoteFields`).
- **Sync** — pushing the local collection to AnkiWeb after a commit so the change reaches the user's
  other devices, via AnkiConnect's `sync` action.
- **Note** — the editable record in Anki: a set of fields plus a note type. ankify creates and edits
  Notes.
- **Card** — what Anki schedules and shows during review, generated from a Note. User-facing copy
  says "cards"; the thing we create/edit is technically a Note.
- **Layout** — how a field's content is arranged vertically: one item per line, with distinct groups
  separated by a blank line, so the card reads cleanly. Distinct from styling (the colour/emphasis that
  encodes meaning). Layout is always applied and never changes the content — only how it's spaced.
- **Note type / model** — the template a Note follows, defining its fields and how cards are
  generated from them. Anki's built-ins include **Basic** (`Front`/`Back`), **Cloze** (`Text` with
  `{{c1::…}}` markup), the reversed and type-in-the-answer Basic variants, and **Image Occlusion**,
  plus any custom types in the user's collection.
- **Deck** — Anki's organizational container for cards. Nesting is expressed with `::`
  (e.g. `Spanish::Verbs`).
- **Fast path** — committing drafts without the review gate, when the user explicitly opts out of
  review for a batch.
- **Profile** — the user's personal, machine-local settings that specialize the generic
  card-quality rubric to them (their subjects and languages, which scripts they can type, styling and
  deck preferences). It is never shipped with the plugin and never committed; with no profile present,
  ankify behaves generically.
- **Steward / curation** — ankify acting on the cards already in the collection to improve them
  (audit and fix), as opposed to authoring new cards from material or chat.
- **Audit** — a read-only pass in which ankify reads the cards in a chosen scope and judges their
  quality against the card-quality rubric. An audit produces a triage; it changes nothing.
- **Scope** — the bounded set an audit runs over: a deck, a tag, or the user's leeches.
- **Triage** — the ranked, read-only summary an audit produces: flagged cards grouped by problem,
  worst-first, shown before any fix is proposed.
- **Leech** — a card Anki has flagged (the `leech` tag) as repeatedly failed. ankify reads it as a
  performance signal pointing at cards worth auditing — not as proof the card is badly written.
- **Fix** — a change an audit proposes for a flagged card: rewrite in place, relayout (re-space the
  field for readability without changing content), split (one card into several), retag, move deck, or
  (only on explicit confirmation) delete.
- **Disposal** — taking an unsalvageable card out of rotation. ankify does this by tagging the card
  for the user's attention in Anki, not by deleting or suspending it directly.
- **`steward::` tags** — ankify's bookkeeping tags on audited cards: `steward::audited` (handled —
  skip on future passes), `steward::keep` (judged hard-but-well-made — excluded permanently), and
  `steward::flagged` (marked for the user's disposal in Anki).
