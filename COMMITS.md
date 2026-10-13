# Commit convention — Manager Library (cohort 3145556)

Applies to all six repositories (`docs`, `api`, `app`, `page`, `infra`, `tool`).
Each repo's seed commit (`Prueba técnica SDD ficha 3145556: …`) predates this convention and is exempt.

## Format

```text
<type>(<scope>): <subject>

[body]

[Refs: T-xx, HU-xx]
```

- One subject line + blank lines as separators. Subject ≤ 72 chars, English, imperative, lowercase first letter, no trailing period.
- `body` is optional: it explains **why**, not what (the diff already says what).
- The `Refs:` footer is required when the commit maps to a `tasks.md` task.

## Types

| Type | Use | Example |
|---|---|---|
| `feat` | New capability or acceptance criterion | `feat(loans): register loan with 1 to 3 lines` |
| `fix` | Fixes existing behavior | `fix(books): reject duplicate ISBN among active books` |
| `refactor` | Restructure without behavior change | `refactor(loans): extract due-date calculation` |
| `test` | Add or adjust tests, no production code | `test(loans): cover 3-book limit per member` |
| `docs` | Documentation only | `docs(php): translate plan to Laravel` |
| `chore` | Maintenance, no product or test changes | `chore(infra): pin Postgres version` |
| `build`, `ci` | Build or continuous integration | `ci(api): check JWT secret in pipeline` |
| `revert` | Revert an earlier commit | `revert(loans): revert double decrement` |

Do not use `wip`, `tmp`, `misc`, `progress`, or uppercase types.

## Scope

Lowercase English (same language as the code), no spaces. It names the module inside the repo, **not** the repo itself:

`auth`, `users`, `books`, `covers`, `members`, `loans`, `overdue`, `report`, `health`, `infra`, `tool`, `app`, `page`, `php`, `contract`, `datamodel`.

Omit it only for cross-cutting changes (`docs: add commit convention`).

## Rules

1. One commit = one verifiable step. Small, frequent commits (partial progress counts toward delivery).
2. One commit touches a single task (`T-xx`). Two tasks = two commits.
3. Every `feat`/`fix` includes or updates its evidence (test, §10 query, walkthrough). Without evidence it is `wip` and stays out of `main`.
4. Never commit secrets (`.env`, hashes, `JWT_SECRET`); never put a `password` in the message.
5. Never mix formatting with behavior: styling goes in a separate `chore` commit.
6. `main` history stays linear and readable: integrate with rebase or fast-forward; `merge` only when it adds context, with its own message.

## Branches

```text
main                      # always deliverable
task/T-09-loan            # one task, short-lived, deleted after merge
fix/CA-03.2-active-isbn   # single fix
```

## Examples

Good:

```text
feat(loans): register loan with 1 to 3 lines

Enforces RN-2 and RN-3: 3-book open limit and overdue block.

Refs: T-09, HU-06
```

```text
test(books): cover deactivation with open loans

Refs: T-08, HU-03
```

```text
docs(php): translate architecture to Eloquent and Laravel
```

Bad (do not do this):

```text
loan progress             # no type or scope
Feat(Loans): Add stuff.   # uppercase type, vague, trailing period
fix: misc                 # "misc" hides two changes
wip                       # a state, not an intent
```

## Verification

Before pushing:

```powershell
git log --oneline -5        # every line matches <type>(<scope>): <subject>
git show --stat HEAD        # a single topic per commit
git status --short          # nothing sensitive in the diff
```
