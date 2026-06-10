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

## Preflight: load the user's profile

After confirming Anki is reachable, read `~/.ankify/profile.md` if it exists. It personalises the
generic rubric — the user's subjects/languages, which scripts they can type (governs whether type-in
cards are appropriate), their styling palette and intensity, and their deck conventions. Let it shape
the type choices, styling, and decks you propose.

If the file does **not** exist, offer once to create it: ask ~3 short questions (what subjects/
languages they study, which scripts they can type, and how plain or styled they like their cards),
then write `~/.ankify/profile.md` from the template at
`${CLAUDE_PLUGIN_ROOT}/references/profile-template.md`. Don't block on it — if they decline, proceed
generically.

## Which flow am I in?

- **Material provided** (PDF, image, file, or a substantial pasted block the user wants carded) →
  *Cards from material*.
- **"Make a card from this" / referencing the conversation** → *Cards from chat*.
- **"Edit / update / fix / change my note about…"** → *Edit existing*.

When ambiguous, ask one short clarifying question rather than guessing.

## Card quality (shared rubric)

Card creation and curation judge cards against the same rubric. **Read it before drafting:**
`${CLAUDE_PLUGIN_ROOT}/references/card-quality-rubric.md`. It is the single source of truth for card
quality — the note-type palette and when each wins, cloze-blank discipline, contrast cards, styling
that encodes meaning, atomic/front-loaded/no-yes-no — do not restate or paraphrase its rules here.
**Choose the note type per card from the rubric's palette** (not just Basic/Cloze) and state the
choice in the review so the user can override.

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

After drafting, **do not commit yet.** Show each draft **the way Anki will test it** — not as raw
field markup — because a flaw like an answer leaking out of a cloze's own example is invisible in raw
`{{c1::…}}` form and obvious once rendered. Before showing, **audit each draft against the rubric
yourself** and self-flag it.

For each draft show:

- A **`✓` or `⚠`** flag (`⚠` if it trips any rubric rule), the **note type**, and target deck.
- The **rendered prompt** with the hidden span shown hidden (e.g. `native counter [ ... ] (한 달…)`)
  → the **answer**. For multi-cloze notes, show each generated card's prompt/answer.
- A one-line **`Tests:`** — the single thing the card tests.
- For a `⚠`, one line naming the problem (e.g. *"leak — 한 달 in the prompt contains the answer 달"*).

Keep clean (`✓`) cards terse; spend the extra explanation line only on `⚠` cards so the user's
attention goes straight to them. Show tags compactly. Example shape:

```
⚠ Cloze · Counters
   Q:  native counter [ ... ] (한 달, 두 달)   A: 달
   Tests: the native month counter
   ⚠ leak — "한 달" in the prompt contains the answer 달
✓ Basic (and reversed) · Vocab
   처음  ⇄  first; the first time
   Tests: the word 처음, both directions
```

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

## Sync

After a successful commit or edit, push the change to AnkiWeb so it reaches the user's other devices.
Call the server's **`sync`** tool, then add a brief line to your report: `Synced to AnkiWeb.`

- **Only sync when something actually changed.** Skip the sync if nothing landed — `addNotes`
  returned all `null` (every slot a duplicate), the user culled everything, or an edit didn't take.
  Don't show the sync line in that case.
- **Fail gracefully.** `sync` uses the AnkiWeb credentials saved in Anki's preferences. If it errors
  — commonly the account isn't logged in, or there's a conflict needing a one-off manual full sync —
  **do not present the card add/edit as failed.** The notes are already in the local collection.
  Report that they landed, then add a calm one-liner that the AnkiWeb sync didn't go through (with the
  reason) and suggest syncing manually in the Anki app.

## Cards from material — flow

1. Preflight.
2. Read the material; extract the test-worthy facts.
3. Draft atomic cards, choosing note type per item; resolve fields, deck, tags; check duplicates.
4. Review gate → commit `addNotes` → sync → report.

## Cards from chat — flow

1. Preflight.
2. Identify the fact(s) the user wants carded from the conversation.
3. Draft (usually 1–few cards); resolve note type/fields/deck/tags.
4. Review gate → commit → sync → report.

## Edit existing — flow

1. Preflight.
2. Find the target note: `findNotes` with an Anki search query built from the user's description;
   if multiple match, show candidates (`notesInfo`) and confirm which one.
3. Show current field values, propose the change, confirm.
4. `updateNoteFields` (warn re: open Browser) → confirm the change landed → sync.
