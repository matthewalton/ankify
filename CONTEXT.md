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
- **Note** — the editable record in Anki: a set of fields plus a note type. ankify creates and edits
  Notes.
- **Card** — what Anki schedules and shows during review, generated from a Note. User-facing copy
  says "cards"; the thing we create/edit is technically a Note.
- **Note type / model** — the template a Note follows. **Basic** (fields `Front`/`Back`) or **Cloze**
  (field `Text` with `{{c1::…}}` markup), plus any custom types in the user's collection.
- **Deck** — Anki's organizational container for cards. Nesting is expressed with `::`
  (e.g. `Spanish::Verbs`).
- **Fast path** — committing drafts without the review gate, when the user explicitly opts out of
  review for a batch.
