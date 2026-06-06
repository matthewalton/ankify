---
description: Find an existing Anki note and update its fields or tags, with confirmation before the change is applied.
argument-hint: "[description of the note to edit and what to change]"
---

Edit an existing Anki note using the **anki-cards** skill.

Request: $ARGUMENTS

Follow the anki-cards skill's *edit existing* flow: run the AnkiConnect preflight, locate the note
with `findNotes` (confirm which one if several match), show me its current field values, propose the
change, and apply it with `updateNoteFields` only after I confirm. Remind me to close Anki's Browser
window if the update doesn't take.
