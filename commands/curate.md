---
description: Audit and improve existing Anki cards in a deck, tag, or your leeches — diagnose against the quality rubric, then fix with review.
argument-hint: "[deck name, tag, or 'leeches' to audit]"
---

Curate existing Anki cards using the **anki-curate** skill.

Scope: $ARGUMENTS

If `$ARGUMENTS` names a deck, audit that deck; if it's a tag, audit that tag; if it's empty or says
"leeches", audit cards tagged `leech`.

Follow the anki-curate skill: run the AnkiConnect preflight, read the cards in scope (excluding the
`steward::audited` / `steward::keep` / `steward::flagged` cards), diagnose each against the
card-quality rubric, then **show a read-only triage before proposing any change**. Apply fixes only
after I approve; deletes need my explicit per-card confirmation. Skip the review gate only if I say so.
