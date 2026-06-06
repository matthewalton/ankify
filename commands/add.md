---
description: Create Anki cards from material (PDF/image/text) or from this conversation, with a review step before they're added.
argument-hint: "[path to material, or a description of what to card]"
---

Create Anki flashcards using the **anki-cards** skill.

Source: $ARGUMENTS

If `$ARGUMENTS` names a file or material, draft cards from it. If it's empty or refers to the
conversation (e.g. "what we just discussed", "this fact"), draft cards from the chat context.

Follow the anki-cards skill: run the AnkiConnect preflight, draft atomic cards (picking Basic vs
Cloze per item), resolve the target deck and tags, check for duplicates, then **show the drafts for
review before committing**. Only skip the review gate if I explicitly say so.
