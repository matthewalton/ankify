# Card-quality rubric

The single source of truth for what makes a good Anki card. Both flows reference this one file:
**`anki-cards`** writes to it, **`anki-curate`** judges existing cards against it. Keep the rules
here and nowhere else so the two flows can never drift apart.

Cards are reviewed hundreds of times — a bad card is worse than no card. A well-made card:

- **Atomic / minimum information.** One fact per card. Split compound facts into separate cards.
- **Uses the right note type** (and states which, when drafting, so the user can override):
  - **Basic** (Front/Back) for discrete Q&A facts you want to recall cold.
  - **Cloze** for facts embedded in context, definitions, and lists. Use `{{c1::hidden}}` markup;
    number deletions (`{{c1::…}}`, `{{c2::…}}`) when more than one piece should be tested separately.
- **Is not a yes/no or trivially-guessable card.** Prompts must be specific and unambiguous.
- **Keeps the front short and the answer precise.** Front-load the cue.
- **Preserves the user's wording/terminology** from the source where it matters for recall.
