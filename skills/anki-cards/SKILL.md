---
name: anki-cards
description: Create or edit Anki flashcards via AnkiConnect. Use when the user gives material (PDF, image, pasted text) to turn into cards, asks to "make/add Anki cards" or "ankify" something, wants a card made from the current conversation, or asks to edit/update/fix an existing Anki note. Drafts cards, shows them for review, then commits to Anki.
---

# Anki card authoring

This skill turns source material or conversation into well-formed Anki notes, and edits existing
notes, using the `anki` MCP server (a wrapper over the AnkiConnect add-on). **The MCP server does no
PDF parsing, OCR, or content understanding — you do.** You read the material, write the card content
(including any Cloze markup), and the server only stores media and creates/updates notes.

## Preflight: is Anki reachable?

Before drafting or committing, confirm AnkiConnect is up by listing decks (the `deckNames` action /
the server's deck-listing tool). If it fails:

> Anki doesn't seem to be reachable. Make sure the Anki desktop app is **open**, with the
> **AnkiConnect** add-on installed (Tools → Add-ons → Get Add-ons → code `2055492159`, then restart
> Anki). It listens on `http://localhost:8765`.

Do not attempt to create or edit cards until it responds.

## Which flow am I in?

- **Material provided** (PDF, image, file, or a substantial pasted block the user wants carded) →
  *Cards from material*.
- **"Make a card from this" / referencing the conversation** → *Cards from chat*.
- **"Edit / update / fix / change my note about…"** → *Edit existing*.

When ambiguous, ask one short clarifying question rather than guessing.

## Card-authoring rules (apply to all creation flows)

Cards are reviewed hundreds of times — a bad card is worse than no card. Follow these:

- **Atomic / minimum information.** One fact per card. Split compound facts into separate cards.
- **Pick the note type per item** (and state which you chose in the review):
  - **Basic** (Front/Back) for discrete Q&A facts you want to recall cold.
  - **Cloze** for facts embedded in context, definitions, and lists. Use `{{c1::hidden}}` markup;
    number deletions (`{{c1::…}}`, `{{c2::…}}`) when more than one piece should be tested separately.
- **No yes/no or trivially-guessable cards.** Prompts must be specific and unambiguous.
- **Keep the front short and the answer precise.** Front-load the cue.
- **Preserve the user's wording/terminology** from the source where it matters for recall.

### Resolve the note type's fields before drafting

Don't assume field names. For the chosen model, look up its fields (the server's
`modelFieldNames` / model-info tool) so you populate the correct keys — e.g. Basic uses
`Front`/`Back`; Cloze uses `Text` (and optional `Back Extra`). If the user has custom note types,
list models (`modelNames`) and ask which to use when unclear.

### Resolve the deck

1. List existing decks (`deckNames`).
2. Suggest the best-matching deck for the material's topic, or propose a new deck name (use `::` for
   nesting, e.g. `Spanish::Verbs`).
3. Confirm with the user in the review step. Only call `createDeck` for a genuinely new deck, after
   confirmation.

### Images and other media

If the material is an image, or a card benefits from one:

- Store the file first (the server's `storeMediaFile`, via base64 `data`, a local `path`, or a `url`),
  then reference it in the field with HTML: `<img src="filename.png">`.
- Or use `addNote`'s inline `picture` array (each entry: `filename` + one of `data`/`path`/`url`, and
  the `fields` to append the `<img>` to).
- Audio renders via `[sound:filename]`.

### Avoid duplicates

Before committing a batch, search for likely existing notes (`findNotes` with an Anki query, e.g.
`deck:"Spanish::Verbs" front:*…*`). Keep Anki's default duplicate rejection **on** so re-running the
same source doesn't double-add. Report anything skipped as a duplicate.

### Tagging

Apply a useful tag so cards are findable later — a source tag (e.g. `source::<filename-or-title>`)
and/or a topic tag. Show the tags in the review so the user can change them.

## The review gate (default behavior)

After drafting, **do not commit yet.** Present the drafts compactly so the user can scan them:

- For each: note type, the field content (Front/Back, or the Cloze `Text`), target deck, and tags.
- A short table or numbered list is ideal.

Then ask the user to **approve / edit / cull**. Apply their edits and re-show if substantial.

**Fast path:** if the user says something like "just add them" / "skip review" / "I trust it", skip
the gate and commit directly. Honor that for the current batch only.

## Commit

- **Creating:** batch with `addNotes`. It returns a note ID per slot, with **`null` for slots that
  failed** (e.g. duplicates) rather than erroring the whole batch. Report the result honestly:
  "Added N cards to <deck>; M skipped as duplicates" — never claim success for `null` slots.
- **Editing:** `updateNoteFields` with the note's `id` and changed `fields` (and tags via the
  appropriate add/remove tag tools). **Caveat:** the update silently fails if that note is currently
  open in Anki's Browser window — warn the user to close it if an edit doesn't take.

## Cards from material — flow

1. Preflight.
2. Read the material; extract the test-worthy facts.
3. Draft atomic cards, choosing note type per item; resolve fields, deck, tags; check duplicates.
4. Review gate → commit `addNotes` → report.

## Cards from chat — flow

1. Preflight.
2. Identify the fact(s) the user wants carded from the conversation.
3. Draft (usually 1–few cards); resolve note type/fields/deck/tags.
4. Review gate → commit → report.

## Edit existing — flow

1. Preflight.
2. Find the target note: `findNotes` with an Anki search query built from the user's description;
   if multiple match, show candidates (`notesInfo`) and confirm which one.
3. Show current field values, propose the change, confirm.
4. `updateNoteFields` (warn re: open Browser) → confirm the change landed.
