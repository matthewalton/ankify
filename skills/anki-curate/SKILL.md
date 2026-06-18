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

## Preflight: load the user's profile

After confirming Anki is reachable, read `~/.ankify/profile.md` if it exists, and pass it to the
Auditor (below) so it judges by the user's preferences (e.g. their styling palette, or that a
hard-but-well-made card in their subject should be kept). With no profile, the audit runs generically.

## Resolve the scope (one scope per audit)

An audit runs over exactly **one scope**. Resolve it interactively from what the user asked — this
stays in the main thread because it may need a clarifying question:

- **Deck** — `deck:"<Deck::Sub>"`. If the user named a deck loosely, list decks first (`listDecks`)
  and confirm which one.
- **Tag** — `tag:<tag>`. Use `getTags` to disambiguate if the tag is unclear.
- **Leeches** (the **default** when the user gives no scope) — `tag:leech`.

**Always exclude already-handled cards.** Append the steward exclusion set to the scope query so past
passes aren't re-triaged:

```
-tag:steward::audited -tag:steward::keep -tag:steward::flagged
```

e.g. `deck:"Spanish::Verbs" -tag:steward::audited -tag:steward::keep -tag:steward::flagged`.

## Delegate the audit to the Auditor subagent

Hand the **reading and diagnosis** to the read-only **`anki-auditor`** subagent (via the Agent tool).
The audit is token-heavy (up to ~50 cards' fields and stats) and must never mutate the collection —
the Auditor has **read-only Anki tools only**, so isolating it there keeps your main thread clean and
makes "the audit cannot change anything" a property of the tooling, not a rule to remember.

Pass it: the resolved scope query, the path to the card-quality rubric
(`${CLAUDE_PLUGIN_ROOT}/references/card-quality-rubric.md`), and the profile path
(`~/.ankify/profile.md`) if present. It reads the scope, applies the two-signal diagnosis, and
**returns the structured triage** — flagged cards grouped by problem, worst-first, each with its
rendered content, verdict, and proposed fix. It changes nothing.

## Triage (read-only)

Present the triage the Auditor returned — this step changes nothing in Anki.

- Groups of flagged cards by problem, **worst-first**.
- For each card: the rendered content (how it's tested), the verdict, and the proposed fix.
- A compact table or grouped numbered list is ideal.

Then ask the user which group(s) or individual cards to act on. Only the selected subset goes through
the review gate. (Verdicts and the fix each maps to are defined in the Auditor; the fixes you apply
below are: rewrite-in-place, split, retag, move-deck, and — only on explicit confirmation — delete. A
`cramped` card's **relayout** is a content-preserving rewrite-in-place — only whitespace markup
(`<br>`) changes — so it is well suited to the fast path.)

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
