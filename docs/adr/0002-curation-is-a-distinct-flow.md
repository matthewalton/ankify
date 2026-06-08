# 0002 — Curation is a distinct flow; extract the shared card-quality rubric

- Status: accepted
- Date: 2026-06-06

## Context

ankify v1 only *authors* cards (from material, from chat) and edits a single named note. The next
capability is **curation**: reading the cards already in the collection, diagnosing their quality,
and fixing the bad ones — ankify acting as a steward of the collection, not just a factory for it.

Curation judges existing cards against the **same rubric** the authoring flow uses to write them
(atomic, unambiguous, front-loaded, no yes/no, well-formed Cloze). That rubric currently lives inline
in `skills/anki-cards/SKILL.md`, and [ADR-0001](0001-depend-on-ankimcp-server.md) establishes that
the methodology — not the AnkiConnect plumbing — *is* the plugin's value. So the rubric is the one
asset that must not be duplicated and allowed to drift.

Two packaging questions follow: (a) does curation grow the `anki-cards` skill or get its own, and
(b) where does the shared rubric live so both flows reference one copy.

Constraints discovered while scoping the flow:

- The bundled `@ankimcp/anki-mcp-server` does **not** expose scheduling mutations — no `suspend`,
  `forgetCards`, `setDueDate`, or `changeModel`. It does expose `updateNoteFields`, `addNotes`,
  `changeDeck`, the tag actions, `deleteNotes`, and the `leech` tag via `findNotes`/`notesInfo`.
- Per-card performance numbers (lapses, ease, interval) are not cleanly readable in bulk; the
  reliable performance signal is the **leech tag** plus deck-level aggregates (`deckStats`,
  `review_stats`).

## Decision

1. **Curation is a separate skill** (`anki-curate`), not a new mode bolted onto `anki-cards`.
   *Create* and *improve* are different verbs with different triggers and flows; separating them
   keeps each skill's activation sharp.

2. **Extract the card-quality rubric** out of `anki-cards/SKILL.md` into a single plugin-root
   reference that both `anki-cards` and `anki-curate` load. One home for the crown-jewel methodology.

3. **The curation flow is:** `preflight → select & read (scoped to a deck / tag / leeches) →
   diagnose → triage → review gate → commit → sync` — reusing the existing review gate, commit
   tools, and sync.

4. **Diagnosis is two-signal, content-adjudicated.** Performance (leech tag + deck aggregates)
   *prioritizes* which cards to look at; content judgment against the rubric *decides the verdict*.
   "Hard, not badly written" is a first-class outcome, not an edge case.

5. **Mutation is conservative and reversible-by-default.** Rewrite-in-place, split (repurpose the
   original note + add the remainder as new cards), retag, and move preserve scheduling history.
   Delete requires explicit per-card confirmation. **Disposal of an unsalvageable card is done by
   tagging it for the user's attention in Anki** — because suspend/forget are not exposed, and
   keeping destructive scheduling ops in the Anki app (behind its undo stack) is safer anyway.

6. **Review scales via triage-then-detail.** A read-only triage (flagged cards grouped by problem,
   worst-first) precedes any proposed change; the detail gate then runs only on the user's selected
   subset, in capped batches. `steward::` tags give the pass memory: `steward::audited` (handled,
   skip on re-runs) and `steward::keep` (judged hard-but-well-made, excluded permanently).

## Consequences

- **Upside:** the rubric has one source of truth; the two verbs activate cleanly; the steward ships
  on today's tool surface with no dependency change; destructive scheduling stays in Anki where it
  has undo.
- **Downside:** in-place Basic↔Cloze conversion isn't possible (it becomes delete-and-recreate,
  resetting history); disposal is a hand-off (tag now, suspend later in Anki) rather than one step;
  the performance signal is coarse (leech + aggregates, not per-card lapse counts).
- **Escalation path (deferred):** if the tag hand-off becomes real friction, swap or augment the MCP
  dependency to expose `suspend`/`forgetCards`/`changeModel` — the cheap, isolated reversal that
  ADR-0001 already anticipated.
- **Deferred scope:** full-collection discovery sweeps, and the authoring-depth fast-follow (Image
  Occlusion, custom note types with bespoke CSS, cloze-overlapping).
