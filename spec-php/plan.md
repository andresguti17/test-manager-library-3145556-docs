# Technical plan — Manager Library (PHP)

How what [`spec.md`](spec.md) and [`constitution.md`](constitution.md) say is built. The schema is **not** described here: it lives in
[`data-model.md`](data-model.md). The shape of the system, in [`architecture.md`](architecture.md).

## 1. Technical decisions

| # | Decision | Choice (PHP) | Why |
|---|---|---|---|
| D-01 | Language and runtime | PHP 8.4 + Laravel 12 | Challenge stack |
| D-02 | HTTP framework | Laravel (API routes + Form Requests + Resources) | Contract in [`api-contract.md`](api-contract.md) |
| D-03 | Persistence | Eloquent + Laravel migrations over PostgreSQL 16 | The schema belongs to the migrations (ADR-001) |
| D-04 | Concurrency | Version witness per book + range `CHECK` ([ADR-002](adr/adr-002-concurrencia-optimista.md)) | Last copy (RNF-2) |
| D-05 | Dates | UTC instants (`timestamptz`); the clock is a port (`Clock`) | Test due dates without waiting 14 days |
| D-06 | Reporting | One aggregate query behind a read port ([ADR-004](adr/adr-004-reporte-agregado-y-congelado.md)) | Stable and cheap |
| D-07 | Value objects | `Isbn`, `Quantity`, `DocumentNumber`, `DateRange`: live inside their owner's row, no table | They have no identity |
| D-08 | Covers | Binary in external storage (disk locally, S3 in production); the database only stores `cover_key` | Opaque key, never a path |
| D-09 | Passwords | Laravel hash (bcrypt) behind the `PasswordHasher` port; the domain never sees the plaintext password | Article IX |
| D-10 | Bootstrapping | The application creates the initial admin from `ADMIN_USERNAME` / `ADMIN_PASSWORD` | The database cannot produce the hash |
| D-11 | Authentication | JWT HS256, 8 hours, key in `JWT_SECRET` | Simple and stateless |
| D-12 | Tests | Pest (over PHPUnit), with fakes for the ports and a real database only in integration tests | Test pyramid |

## 2. Environment configuration

| Variable | Required | Example |
|---|---|---|
| `DATABASE_URL` | yes | `postgresql://library:library@db:5432/library` |
| `JWT_SECRET` | yes | long random string |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | yes | defined in `.env`, never in the repo |
| `COVERS_DIR` | no | `./var/covers` |
| `CORS_ORIGIN` | yes | `http://localhost:5173` |

## 3. Test strategy

| Level | What it tests | Against what |
|---|---|---|
| Domain | Invariants: max 3, 14-day due date, available in range | Nothing: pure objects |
| Application | Use cases: lend, return, report | Port fakes |
| Integration | Repositories and report query | Real PostgreSQL |
| Contract | Single error body and status codes | The running API |
| Architecture | Articles I, II and III | `deptrac` + test in `tests/Architecture/` |
| Concurrency | Two loans of the last copy | Real PostgreSQL |

## 4. Build order

Skeleton and health → initial migration with seed → domain → use cases with fakes → adapters → API → contract and concurrency tests.
The detail, with evidence, is in [`tasks.md`](tasks.md).
