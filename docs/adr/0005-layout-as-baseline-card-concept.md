# 0005 — Layout is a distinct, always-on card concept

- Status: accepted
- Date: 2026-06-18

## Context

Card fields were often written as a single run-on line — definitions, examples, and notes separated
only by dots, slashes, or hyphens. The content was frequently correct (a well-formed contrast, an
atomic fact with a supporting example), but it read as a cramped wall on a phone.

The rubric already had a **"Styling encodes meaning"** section, but that is specifically about
*colour/emphasis* — colour-coding the two sides of a contrast, demoting an example, highlighting the
changed part of `X → Y`. It said nothing about *vertical arrangement*. There was also a real risk of
collision with the **atomic / contrast** rules: "this card has several parts, break them onto lines"
can be misread as "this card has several ideas, split it."

So the question was how to name and place the readability concern, whether it should be a user-tunable
preference, how to encode it, and how to retrofit it onto an existing collection.

## Decision

1. **Layout is its own concept, distinct from styling.** It gets its own glossary term (`CONTEXT.md`)
   and its own rubric section. Styling = colour/emphasis that *encodes meaning*; layout = how content
   is *arranged vertically*. Keeping them separate stops either rule from being overloaded.

2. **Layout never changes content; atomicity still decides card boundaries.** Layout only arranges what
   is already on the card. It never merges separate ideas to dodge a split and never rescues a
   non-atomic card — the atomic / contrast rules are unchanged and decide first.

3. **A two-level model:** one item per line within a group, and distinct groups separated by a blank
   line. What counts as a group is a per-card judgment, not a fixed taxonomy.

4. **Encoded inline as `<br>`, no CSS.** Line break = `<br>`; section gap = `<br><br>`. This renders
   identically across devices and is safe on note types ankify didn't create — consistent with the
   existing "never edit a shared note type's CSS" rule.

5. **Always-on baseline, orthogonal to styling intensity.** Layout applies to every card regardless of
   the `minimal | moderate | rich` styling-intensity knob and regardless of whether a profile exists.
   It is readability, not decoration; the intensity knob governs colour only.

6. **Rolled out through the existing curation flow, not a new one.** The read-only `anki-auditor`
   ([ADR-0004](0004-readonly-auditor-subagent.md)) gains an orthogonal `cramped` flag that can sit
   alongside an otherwise `well-made` content verdict. `/ankify:curate` fixes it as a rewrite-in-place
   via `updateNoteFields` (scheduling history preserved). Because the fix is content-preserving, it is
   eligible for the existing fast-path.

Because both `/ankify:add` and `/ankify:curate` read the one shared rubric, this single rule governs
new cards *and* lets curation retrofit old ones — no per-skill change needed.

## Consequences

- **Upside:** cards read cleanly on phones; the standard lives in one place and applies to authoring
  and curation without drift; the retrofit reuses the conservative, reviewed, reversible curate
  machinery rather than a bespoke bulk operation.
- **Considered and rejected:** folding layout into the "styling" section (overloads one name with two
  ideas); gating layout behind the styling-intensity knob (would leave "minimal" users with the cramped
  cards we set out to fix); a dedicated whole-collection relayout pass outside curation (duplicates
  ADR-0002/0004 machinery); preferring layout over splitting non-atomic cards (would weaken the atomic
  rule the rubric treats as central).
- **Downside:** a whole-collection retrofit is a large, effectively one-way bulk edit (mitigated:
  `updateNoteFields` preserves scheduling history, and it changes only whitespace markup). The auditor
  now carries an extra, orthogonal axis (`cramped`) on top of the content verdicts.
- **Re-sweep gap (and the rule it implies).** Curation normally excludes `steward::audited` cards so
  past passes aren't re-triaged — but those cards were judged against the rubric *as it stood then*.
  Introducing a new quality dimension (here, layout) leaves every already-audited card unchecked on
  that dimension. So: **whenever a new dimension is added to the rubric, run one bulk re-sweep that
  ignores the steward exclusions, scoped to the new dimension only** (content verdicts already stand).
  This is a one-time migration, not a change to the normal exclude-audited behaviour.
