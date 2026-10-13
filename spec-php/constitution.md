# Constitution — Manager Library (PHP)

Non-negotiable principles of the project. They are not style or preference: each article states **how it is verified**, because a principle
nobody can check is an intention. A change that violates an article is rejected without discussing the rest.

---

## Article I — The domain depends on nothing

`app/Library/Domain/` imports no other project module and no third-party libraries: only standard PHP (`readonly class`, `DateTimeImmutable`, native types); no Eloquent, no Laravel, no framework validation.

**Why:** it is the only thing that allows testing the invariants without a database, without a server, and without a container, and switching ORMs without touching rules.
**Verification:** a `deptrac` contract and an architecture test in `tests/Architecture/` forbid the domain from depending on `Application`, `Adapters`, or `Bootstrap`.

## Article II — The core declares its ports; the world implements them

The application layer only knows the domain. Everything it needs from the outside (repositories, clock, password hashing, cover
storage, token issuance) it expresses as a **port that it defines itself**. No framework or ORM type appears in a port signature.

**Verification:** `deptrac` prevents `Application` from importing from `Adapters`. Application tests run with fakes and without referencing `Adapters`.

## Article III — A single composition point

A single module (`app/Library/Bootstrap/Composition/PortBindings.php`) is where a port meets its adapter. Outside of it, no infrastructure is instantiated.
**Verification:** no instantiation of an `adapters` class outside that module; checked with an architecture test.

## Article IV — SOLID is not decorative

| Principle | Concrete application | Verification |
|---|---|---|
| **SRP** | One application service = one use case. `LendBooksService` lends and does nothing else | A service with two reasons to change is split |
| **OCP** | Changing cover storage (disk → S3) does not touch domain or application | The new adapter only adds one line in the composition |
| **LSP** | Test fakes replace the real adapter without changing expectations | The same test passes against the fake and the real one |
| **ISP** | Small ports: `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage` | No implementer leaves methods unused |
| **DIP** | The port is defined by whoever consumes it | Ports live in the application package |

## Article V — The schema belongs to the migrations

The `infra` repository brings up an **empty** engine. All DDL (tables, indexes, constraints, extensions, and seed) lives in the
application migrations (Laravel migrations). See [ADR-001](adr/adr-001-propiedad-del-esquema.md).
**Verification:** `infra` contains no `CREATE TABLE`.

## Article VI — Invariants the database can enforce, the database enforces

A rule that only lives in code protects the application, not the data. Every invariant expressible in the engine (`CHECK`, `UNIQUE`, FK)
is declared there. Anything not enforced is marked **pending** in [`data-model.md`](data-model.md); it is not promised.
**Verification:** the queries in `data-model.md` §10 match what the document declares.

## Article VII — Derived values are computed, not stored

The state of a loan (`open`, `overdue`, `closed`) and the days of delay are computed. There is no column for them.
**Verification:** `data-model.md` lists no `status` or `days_late` column.

## Article VIII — Frozen values do not follow the catalog

The loan line stores the title, ISBN, and category at the moment of the loan. No report reads them from the live catalog.
**Verification:** a test that renames a book after lending it and checks that the report keeps the old title ([ADR-004](adr/adr-004-reporte-agregado-y-congelado.md)).

## Article IX — No secret enters the repository or a log

Neither passwords, nor precomputed hashes, nor the token key. Every secret comes from the environment. The real `.env` is in `.gitignore`; only `.env.example` is versioned.
**Verification:** `gitleaks` or a pattern `grep` in CI; no HTTP response includes `password` or `password_hash`.

## Article X — The engine wins

If the data document and the database disagree, the document is broken and the document is fixed.

## Article XI — Language

Code, columns, and routes in English. Documentation in English; user-facing messages in Spanish.

## Article XII — Nothing is taken for granted without evidence

A task in [`tasks.md`](tasks.md) is done when its **evidence** exists and can be executed, not when "it is already coded".
