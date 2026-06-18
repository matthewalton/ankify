# Card-quality rubric

The single source of truth for what makes a good Anki card. Both flows reference this one file:
**`anki-cards`** writes to it, **`anki-curate`** judges existing cards against it. Keep the rules
here and nowhere else so the two flows can never drift apart.

Cards are reviewed hundreds of times — a bad card is worse than no card.

## Load the user's profile first

If `~/.ankify/profile.md` exists, read it before drafting or judging. It answers the open questions
this rubric leaves to the person — their subjects/languages, which scripts they can type (governs
type-in eligibility), their styling palette and intensity, and their deck conventions. With no
profile present, behave generically and use restrained defaults.

## Decide what to test *before* how to test it

Every card exists to test **one** thing. Before choosing a note type or placing a blank, write the
one-line answer to *"what is this card testing?"* If you can't state it in a phrase, the card isn't
ready. This single line is also what you show at the review gate (`Tests: …`), and it is the yardstick
every rule below is measured against.

## The note-type palette — pick the form that fits the test

Don't default to Basic/Cloze. Choose the type that matches what's being tested, and **state the chosen
type at review so the user can override.** Resolve the model's field names before drafting (the
server's `modelFieldNames` / model-info tool) — e.g. Basic uses `Front`/`Back`; Cloze uses `Text`
(and optional `Back Extra`).

- **Basic** (`Front`/`Back`) — recall one discrete fact in **one** direction.
- **Basic (and reversed card)** — a pair where you need **both** recognition *and* production (most
  vocabulary). Generates both directions from one note.
- **Basic (type in the answer)** — when the **exact form or spelling is the point** and typing it
  forces the precision. Only when the user can type that script (check the profile).
- **Cloze** (`Text`) — a fact embedded in a sentence, a **contrast**, or one specific element of a
  pattern that you want to test in place.
- **Image Occlusion** — **spatial / visual / labeled / tabular** material: diagrams, maps, charts,
  conjugation tables, labeled images.

Custom note types (`createModel`) are deliberately out of scope — reach for one only when nothing
above fits, and say why.

## Cloze discipline — where the blank goes

Most bad cloze cards fail here, not in the note type.

1. **Blank the hard/novel element, never the trivial scaffolding.** Test the thing being learned, not
   the glue around it. (A card that hides a common helper word while handing over the new grammar is
   testing the wrong half.)
2. **The prompt may not contain its own answer.** Scan the visible context — examples, repeated
   tokens, the rest of the sentence — for anything that reveals the hidden span. If an example gives it
   away, move that example to `Back Extra`, or replace the giveaway with a controlled hint:
   `{{c1::answer::hint}}`.
3. **One idea per cloze number.** Different numbers (`{{c1::…}}`, `{{c2::…}}`) are tested as separate
   cards; the same number is revealed together. Never reuse a number across different answers, and
   never group two unrelated facts under one.
4. **Keep the prompt clean.** Mnemonics, examples, and the "why" go in `Back Extra`, off the prompt
   side where they can't leak.

## Contrast cards — a named pattern, with guardrails

When the whole point is telling two things apart (A vs B, can vs can't, native vs borrowed form),
testing the **distinction** on one card is correct and often better than splitting it. This is allowed
**only** with guardrails:

- Each side gets its **own cloze number**, so they're tested independently.
- **Neither side may leak the other** (rule 2 above still holds).
- The contrast must be the actual learning target — if the two sides are unrelated facts that merely
  sit near each other, that's not a contrast, it's a non-atomic card: **split it.**

## Atomic / minimum information

One testable idea per card. A contrast (above) counts as one idea. Split genuinely compound facts into
separate cards.

## Styling encodes meaning — not decoration

Visual structure earns its place when it makes the test clearer; colour for its own sake is noise.

- **Inline HTML/CSS in the fields only** — self-contained, travels with the card, works on every
  device.
- **Style semantic roles**, not arbitrary words: the **contrast-axis** (colour-code the two sides of a
  comparison), a **hint**, an **example** (visually demoted below the prompt), a **transform-target**
  (emphasise the part that changes in `X → Y`). A two-axis contrast or a small table often reads far
  better than a run-on sentence.
- **Never edit the CSS of a note type ankify didn't create.** Restyling the shared Basic/Cloze models
  silently changes the user's other cards. Keep all styling inline.
- **Restraint by default.** The profile sets the palette and intensity (including "minimal" for people
  who want plain cards). With no profile, keep it plain.

## Layout — let the card breathe

Layout is how a field's content is arranged vertically. It is separate from styling (colour/emphasis
above): layout never changes, adds, or removes content — it only arranges what's already there. Card
boundaries are still decided by the atomic / contrast rules above; layout never merges separate ideas
to avoid a split, and it never rescues a non-atomic card.

Apply it always. Layout is readability, not decoration, so it is baseline for **every** card —
regardless of styling intensity (including "minimal") and whether a profile exists. The intensity knob
governs colour/emphasis only, never whether a card is laid out.

Two levels:

- **One item per line.** Within a group, give each discrete piece its own line — each definition, each
  example sentence, each form. Don't run them together with dots, slashes, or hyphens.
- **Sections with a gap.** When a field holds distinct groups of content (e.g. the definitions vs. the
  example sentences), separate the groups with a blank line so they read as distinct blocks. What
  counts as a group is a per-card judgment — definitions/examples is only one common shape.

**Mechanism — inline only, no CSS.** Use `<br>` for a line break and a blank line (`<br><br>`) for the
gap between sections. This renders identically on phone and desktop and is safe on note types ankify
didn't create (the no-editing-shared-CSS rule above still holds).

**Cloze.** Lay out the `Text` field the same way, but never break the cloze syntax (`{{c1::…}}`) and
never let a line break expose or separate a hidden span in a way that leaks it (rule 2 of cloze
discipline still holds).

## The basics (always)

- **Not yes/no or trivially-guessable.** Prompts must be specific and unambiguous.
- **Front short, answer precise.** Front-load the cue.
- **Preserve the user's wording/terminology** from the source where it matters for recall.
