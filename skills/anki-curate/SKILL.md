---
name: anki-curate
description: Audit and improve the Anki cards already in your collection. Use when the user asks to "curate / clean up / fix / improve my Anki cards", review a deck's card quality, deal with leeches, or audit cards against the quality rubric — as opposed to creating new cards from material. Reads cards in a chosen scope, diagnoses them against the shared rubric, shows a read-only triage, then applies fixes only after review.
---

# Anki card curation (the steward)

This skill is the **steward** of an existing collection: it reads the cards already in Anki, judges
their quality, and improves the bad ones. It does **not** author cards from material or chat — that's
the `anki-cards` skill. Like that skill, it uses the `anki` MCP server (a wrapper over AnkiConnect),
and **the server does no content understanding — you do.** You read each card and decide whether it is
well-made, salvageable, or beyond saving.

## Card quality (shared rubric)

Card creation and curation judge cards against the same rubric. **Read it before diagnosing:**
`${CLAUDE_PLUGIN_ROOT}/references/card-quality-rubric.md`. It is the single source of truth for card
quality — do not restate or paraphrase its rules here. **It is the verdict standard for every
diagnosis below.**

## Preflight: is Anki reachable?

Before reading or changing anything, confirm AnkiConnect is up by listing decks (the `listDecks` /
`deckNames` action). If it fails:

> Anki doesn't seem to be reachable. Make sure the Anki desktop app is **open**, with the
> **AnkiConnect** add-on installed (Tools → Add-ons → Get Add-ons → code `2055492159`, then restart
> Anki). It listens on `http://localhost:8765`.

Do not attempt to read or curate cards until it responds.

## Select & read (scope the audit)

An audit runs over exactly **one scope**. Resolve it from what the user asked:

- **Deck** — `findNotes` with `deck:"<Deck::Sub>"`. If the user named a deck loosely, list decks
  first (`listDecks`) and confirm which one.
- **Tag** — `findNotes` with `tag:<tag>`. Use `getTags` to disambiguate if the tag is unclear.
- **Leeches** (the **default** when the user gives no scope) — `findNotes` with `tag:leech`.

**Always exclude already-handled cards.** Append the steward exclusion set to every scope query so
past passes aren't re-triaged:

```
-tag:steward::audited -tag:steward::keep -tag:steward::flagged
```

e.g. `deck:"Spanish::Verbs" -tag:steward::audited -tag:steward::keep -tag:steward::flagged`.

Then gather what you need to judge:

- **Performance context (for prioritization):** `deckStats` and/or `review_stats` for the deck(s) in
  scope; treat `tag:leech` membership as the per-card performance flag.
- **Card content:** read the notes with `notesInfo` (and `get_cards` where card-level data helps).
- **Cap: read up to ~50 cards in scope for a triage.** If the scope is larger, say so and take the
  highest-priority ~50 first (leeches and low-retention decks first); offer to continue in a later
  pass.

## Diagnose (two-signal, content-adjudicated)

Diagnosis uses two signals, and they do different jobs:

- **Performance prioritizes.** A leech tag, or a card in a low-retention deck (per the aggregates),
  means "look here first." **A leech is a *pointer*, not proof the card is badly written.**
- **Content decides.** The verdict comes from judging the card against the rubric — never from the
  performance signal alone.

Assign each card a verdict:

- **`well-made`** — meets the rubric. Skip (no change); mark `steward::audited`.
- **`hard-but-well-made`** — genuinely difficult but correctly built. A **first-class outcome**, not
  an edge case. Leave the content alone; mark `steward::keep` so it's excluded permanently.
- **`not-atomic`** — tests several facts at once → **split**.
- **`ambiguous` / `yes-no` / `not front-loaded`** → **rewrite in place**.
- **`misfiled`** — wrong deck or missing/wrong tags → **move deck / retag**.
- **`unsalvageable`** — can't be fixed into a good card → **disposal** (flag for the user; see Commit).

## Triage (read-only)

**Before proposing any change, show a read-only triage.** This step changes nothing in Anki.

- Group flagged cards by problem, **worst-first**.
- For each card show: the field content (Front/Back or Cloze `Text`), the verdict, and the proposed
  fix.
- A compact table or grouped numbered list is ideal.

Then ask the user which group(s) or individual cards to act on. Only the selected subset goes through
the review gate.

## Review gate

On the user's selected subset only, **in batches of ~10**:

- Show the concrete **before → after** for each proposed fix (and, for splits, the resulting set of
  cards).
- Ask the user to **approve / edit / cull** per card. Apply their edits and re-show if substantial.

**Fast path:** if the user says "just fix them" / "I trust it", skip the per-card gate for the current
batch. Honor that for that batch only. Deletes are **never** part of the fast path (see Commit).

## Commit (conservative, reversible-by-default)

Map each approved fix to a tool. Prefer changes that **preserve scheduling history**:

- **Rewrite in place:** `updateNoteFields` with the note `id` and changed `fields`. Preserves history.
  **Caveat:** the update silently fails if that note is open in Anki's Browser window — warn the user
  to close it if an edit doesn't take.
- **Split:** `updateNoteFields` to repurpose the **original** note down to one atomic fact (keeping its
  history), then `addNotes` for the remaining facts as new cards (these start fresh).
- **Retag:** `addTags` / `removeTags` / `replaceTags`.
- **Move deck:** `changeDeck`. Preserves history.
- **Delete:** `deleteNotes` — **only after explicit, per-card confirmation.** Never delete in a batch
  default and never on the fast path.
- **Disposal of an unsalvageable card:** do **not** delete or try to suspend it. Add the
  `steward::flagged` tag so the user can suspend/forget it themselves in Anki later. Suspend, forget,
  and set-due are **not exposed** by the MCP server, and keeping those destructive scheduling ops in
  the Anki app (behind its undo stack) is safer anyway. Optionally open the flagged cards for the user
  with `guiBrowse` (query `tag:steward::flagged`).

**Pass memory.** After handling a card, tag it so future passes skip it:

- `steward::audited` — handled (rewritten, split, moved, retagged, or judged well-made).
- `steward::keep` — judged hard-but-well-made; excluded permanently.
- `steward::flagged` — marked for the user's disposal in Anki.

All three are excluded by the scope query above on future runs.

## Sync

After a successful commit, push the change to AnkiWeb so it reaches the user's other devices. Call the
server's **`sync`** tool, then add a brief line to your report: `Synced to AnkiWeb.`

- **Only sync when something actually changed.** If nothing landed (every card was well-made / kept, or
  the user culled everything), don't sync and don't show the sync line.
- **Fail gracefully.** `sync` uses the AnkiWeb credentials saved in Anki's preferences. If it errors
  (commonly: account not logged in, or a conflict needing a one-off manual full sync), **do not present
  the curation as failed** — the changes are already in the local collection. Report that they landed,
  then add a calm one-liner that the AnkiWeb sync didn't go through (with the reason) and suggest
  syncing manually in the Anki app.

## Curation flow — summary

`preflight → scope & read → diagnose → triage (read-only) → review gate → commit → sync`
