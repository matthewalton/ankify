# 0003 — Personalise via a machine-local profile, keep the plugin generic

- Status: accepted
- Date: 2026-06-10

## Context

Deepening the card-authoring expertise ([the rubric enrichment](../../references/card-quality-rubric.md))
surfaced a tension. Good cards depend on facts about the *person*: which languages they study, which
scripts they can actually type (this decides whether "type in the answer" cards make sense), how plain
or styled they want cards, how their decks are organised. But ankify is a plugin anyone installs — none
of those specifics can be baked into the shipped rubric, skills, or `CONTEXT.md` without making the
plugin secretly about one user.

So the question: where do per-user specifics live, such that the plugin stays generic but can still
behave as if it knows *you*?

## Decision

1. **A single machine-local profile file at `~/.ankify/profile.md`.** It holds the user's specifics —
   subjects/languages, typable scripts, styling palette and intensity, deck conventions. A committed,
   generic template ships at `references/profile-template.md`; the user's actual profile lives in their
   home directory, **outside any repo, so it is inherently uncommitted and never ships.**

2. **Read at preflight; bootstrap on first run.** Both `anki-cards` and `anki-curate` read the profile
   after the Anki-reachable check and let it shape type choices, styling, and decks. With no profile,
   they offer once to create it from the template and otherwise **behave generically** — the profile is
   an enhancement, never a requirement.

3. **The rubric asks the questions; the profile answers them.** The rubric stays domain-neutral ("use
   type-in when exact form is the point *and the user can type that script*"); the profile supplies the
   answer ("scripts I can type: …"). The two never merge.

4. **Home-level, not project-level.** The profile follows the user across every project and every piece
   of material they ankify, because their Anki collection is one thing regardless of working directory.
   Project-local overrides are deferred until there's evidence they're needed.

## Consequences

- **Upside:** a clean seam between "plugin strangers install" and "behaves like it knows me." The
  shipped artefacts stay generic and reviewable; personalisation is opt-in and lives where the user can
  edit it by hand.
- **Downside:** a contract on the path `~/.ankify/profile.md` and the template's shape — moving either
  later is a (small) migration for anyone who has a profile. The skills must handle the
  profile-absent case gracefully everywhere they read it.
- **Deferred:** project-local profiles/overrides; any richer schema than a hand-edited markdown file.
