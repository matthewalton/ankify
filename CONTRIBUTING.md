# Contributing to ankify

## Commit message convention

ankify uses [Conventional Commits](https://www.conventionalcommits.org/). Every commit subject is a
single line in the form:

```
type(scope): summary
```

- **One line only.** No body or footer unless genuinely necessary; no `Co-Authored-By` trailers.
- **Imperative mood**, lower-case summary, no trailing period (e.g. "add edit command", not "Added
  edit command.").
- Keep the subject under ~72 characters.

### Types

| Type       | Use for                                                            |
|------------|--------------------------------------------------------------------|
| `feat`     | a new capability or user-facing feature                            |
| `fix`      | a bug fix                                                           |
| `docs`     | documentation only (README, CONTEXT, ADRs, this file)              |
| `refactor` | code change that neither fixes a bug nor adds a feature            |
| `chore`    | maintenance: config, dependencies, manifests, release bumps        |
| `ci`       | CI / workflow changes                                              |
| `test`     | adding or adjusting tests                                          |
| `style`    | formatting/whitespace only, no behavior change                    |

### Scopes

Optional, but preferred. Use the area touched:

`skill`, `commands`, `mcp`, `marketplace`, `plugin`, `docs`, `ci`, `release`.

### Examples

```
feat(skill): add cloze deletion guidance to card-authoring rules
feat(commands): add /ankify:edit command
fix(mcp): correct ANKI_CONNECT_URL default
docs(adr): record decision to depend on @ankimcp server
chore(release): bump to v0.2.0
ci: validate plugin.json required fields
```

## Releases

The marketplace entry is pinned to `main`, so every push to `main` is installable. For a real
release, bump `version` in **both** `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`
in a single `chore(release): bump to vX.Y.Z` commit.
