# Commit convention — Manager Library (cohort 3145556)

Applies to all six repositories (`docs`, `api`, `app`, `page`, `infra`, `tool`).
Each repo's seed commit (`Prueba técnica SDD ficha 3145556: …`) predates this convention and is exempt.

## Format

```text
<type>(<scope>): <subject>

[body]

[Refs: T-xx, HU-xx]
```

- One subject line + blank lines as separators. Subject ≤ 72 characters, English, imperative, lowercase first letter, no trailing period.
- The `body` is optional: explain the **why**, not the what (the diff already says the what).
- The `Refs:` footer is required when the commit belongs to a `tasks.md` task.

## Types

| Type | Use | Example |
|---|---|---|
| `feat` | New capability or acceptance criterion | `feat(loans): register loan with 1 to 3 lines` |
| `fix` | Fix behavior that already existed | `fix(books): reject duplicate ISBN among active` |
| `refactor` | Restructure without behavior change | `refactor(loans): extract due-date computation` |
| `test` | Add or adjust tests, no production code | `test(loans): cover 3-book limit per member` |
| `docs` | Documentation only | `docs(php): translate plan to Laravel` |
| `chore` | Maintenance, no product or test changes | `chore(infra): pin Postgres version` |
| `build`, `ci` | Build or continuous integration | `ci(api): check JWT secret in pipeline` |
| `revert` | Revert an earlier commit | `revert(loans): revert double decrement` |

Do not use `wip`, `tmp`, `misc`, `progress`, or uppercase types.

## Scope

Lowercase English (same language as the code), no spaces. It is the module inside the repo, **not** the repo name:

`auth`, `users`, `books`, `covers`, `members`, `loans`, `overdue`, `report`, `health`, `infra`, `tool`, `app`, `page`, `php`, `contract`, `datamodel`, `commits`.

Omit it only for cross-cutting changes (`docs: adjust commit convention`).

## Rules

1. One commit = one verifiable step. Small, frequent commits (delivery grades partial progress).
2. One commit touches a single task (`T-xx`). Two tasks = two commits.
3. Every `feat`/`fix` includes or updates its evidence (test, §10 query, walkthrough). Without evidence it is `wip` and stays out of `main`.
4. Never commit secrets (`.env`, hashes, `JWT_SECRET`); never a `password` in the message.
5. Never mix formatting with behavior in one commit.
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

Enforce RN-2 and RN-3: cap of 3 open books, block members with overdue items.

Refs: T-09, HU-06
```

```text
test(books): cover retirement with open loans

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
