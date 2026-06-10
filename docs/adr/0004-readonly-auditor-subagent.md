# 0004 — Delegate curation's Audit to a read-only subagent

- Status: accepted
- Date: 2026-06-10

## Context

[ADR-0002](0002-curation-is-a-distinct-flow.md) established curation as a distinct flow whose shape is
`preflight → select & read → diagnose → triage → review gate → commit → sync`, with mutation kept
**conservative and reversible-by-default**. Two properties of the early phases stand out:

- **Select & read + diagnose are token-heavy.** Reading up to ~50 cards' fields plus deck/`review`
  aggregates fills the working context with raw card data that the user never needs to see — only the
  resulting triage matters to them.
- **Those phases must never mutate.** The audit is, by definition, read-only; all change happens later,
  behind the review gate.

Keeping the audit inline in the `anki-curate` skill works, but it bloats the main conversation with
card dumps and leaves "the audit doesn't change anything" as a *rule the model must follow* rather than
something it *cannot* break.

## Decision

1. **A read-only `anki-auditor` subagent performs the Audit.** The `anki-curate` skill resolves the
   scope interactively (it may need a clarifying question), then delegates reading + diagnosis to the
   subagent, which returns a structured, worst-first **triage**. Everything after — triage
   presentation, the review gate, and the Fixes — stays in the main thread where the user approves.

2. **The boundary is enforced by the tool grant, not by convention.** The Auditor's frontmatter grants
   **only read-only Anki tools** (`findNotes`, `notesInfo`, `get_cards`, `listDecks`, `deckStats`,
   `review_stats`, `getTags`) plus `Read`. The mutation tools (`updateNoteFields`, `addNotes`,
   `deleteNotes`, `changeDeck`, the tag actions) are simply absent from it — so the audit *cannot*
   change the collection, making ADR-0002's conservative-mutation stance a property of the tooling.

3. **Same verdict standard as authoring.** The Auditor reads the *same* shared rubric (and the user's
   profile), so curation still judges by the exact standard authoring writes to — the no-drift goal of
   the single-source rubric is preserved across the subagent boundary.

4. **Authoring is not delegated.** `anki-cards` stays inline — drafting is interactive and belongs in
   the conversation. Only curation's bulk read earns a subagent.

## Consequences

- **Upside:** the main thread stays clean of card dumps; "the audit is read-only" is guaranteed by the
  available tools; the conservative, user-gated mutation model from ADR-0002 is unchanged.
- **Downside:** a handoff boundary — the Auditor returns a triage as structured text the skill then
  presents, and it can't ask the user clarifying questions mid-audit (scope resolution stays in the
  main thread to compensate). Two places now describe the verdict vocabulary (the Auditor owns the
  diagnosis detail; the skill references it) — kept in sync by pointing both at the one rubric.
- **Lives at** `agents/anki-auditor.md` (plugin-root convention, alongside `commands/` and `skills/`),
  so it ships with the plugin.
