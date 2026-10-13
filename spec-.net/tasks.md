# Tasks — Manager Library (.NET)

Each task is done when its **evidence** exists and can be run (article XII). Initial status of all: **pending**.

| # | Task | Depends on | Evidence |
|---|---|---|---|
| T-01 | Project skeleton and `GET /health` | — | `GET /health` responds `{"status":"ok"}`; the container starts with one command |
| T-02 | Initial migration: six tables, FK, unique indexes and the five seeded categories | T-01 | The `data-model.md` §10 queries return what is documented; no column with `DEFAULT` |
| T-03 | Domain: `Isbn`, `Quantity`, `DateRange` (value objects) | T-01 | Unit tests: 13-digit ISBN, quantity > 0, end ≥ start |
| T-04 | Domain: `Book` (`Withdraw`, `Restock`, soft delete) | T-03 | Tests: never < 0, never > total |
| T-05 | Domain: `Loan` (`AddItem`, `ReturnItem`, computed status, 14 days, max. 3) | T-04 | Tests with a fake `Clock`: `open`, `overdue`, `closed` |
| T-06 | Domain: `Member` and `User` (name normalization, valid role) | T-03 | `"Ana "` becomes `ana`; `admin` role via API rejected |
| T-07 | Ports and **authentication** use cases (`LoginService`, `CreateLibrarianService`) with fakes | T-06 | HU-01 and HU-02 pass against fakes |
| T-08 | Ports and **book** use cases (create, search, edit, deactivate, cover) with fakes | T-04 | HU-03 and HU-04 pass against fakes |
| T-09 | **Lend** use case (`LendBooksService`) with fakes | T-05, T-08 | Full HU-06: limits, overdue, without copies |
| T-10 | **Return** and **overdue** use case | T-09 | HU-07 and HU-09; returning twice → `ITEM_ALREADY_RETURNED` |
| T-11 | **Most-borrowed report** use case and its read port | T-09 | HU-10; the old title is kept after a rename (ADR-004) |
| T-12 | Persistence adapter: book, member, user repositories | T-02, T-08 | Integration tests against real PostgreSQL |
| T-13 | Persistence adapter: loan repository + version witness | T-12, T-09 | Two loans of the last copy: one 201, the other 409 (RNF-2) |
| T-14 | Security adapter: password hash and JWT token | T-07 | The password never appears in logs or responses (RNF-3) |
| T-15 | Cover adapter (disk) | T-08 | Uploading a 2 MB PNG ok; 3 MB → 422; without a cover → null `cover_key` |
| T-16 | HTTP adapter: [`api-contract.md`](api-contract.md) routes, single error body, role-based authorization | T-10, T-11, T-14, T-15 | Contract test walks all routes and all §2 codes |
| T-17 | Composition and startup (initial admin from environment) | T-16 | The system starts with an empty database and admin login works |
| T-18 | Push the **domain-only** invariants down to the database: `ck_book_total_positive`, `ck_user_role` | T-12 | An `INSERT` via `psql` violating them fails; `data-model.md` §4 updated |
| T-19 | Architecture tests (articles I, II and III) | T-17 | NetArchTest passes and fails if a forbidden import is introduced |
| T-20 | `infra`: `docker compose up` starts the empty engine, the API and the frontend | T-17 | One command; the database starts without tables until the migrations run |
| T-21 | `tool`: demo seeder **through the API** (≥ 40 books, 10 members, 15 loans, 3 overdue) | T-17 | After running it, `/loans/overdue` returns 3 lines and the report has rows |
| T-22 | `app`: login, catalog, members, lend, return, overdue, report | T-16 | Manual end-to-end walkthrough at the fair |
| T-23 | `page`: static public presentation site | — | Opens without the API; correct on mobile and desktop |

## Test done definition

Whoever reaches T-17 with evidence has the **core** done. T-18 through T-23 are what separates good work from outstanding work. Performance is
evaluated with SDD, not task count: one task with evidence is worth more than three half-done ones.
