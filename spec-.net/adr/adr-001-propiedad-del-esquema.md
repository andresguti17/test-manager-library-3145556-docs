# ADR-001 · The schema belongs to the application migrations

- **Status:** accepted
- **Context:** the `infra` repository starts a database engine. Someone could put the DDL in a container startup
  script and someone else in the application migrations; with two owners, nobody knows which schema runs in production.
- **Decision:** **all DDL lives in the application migrations** (tables, indexes, constraints and the category seed).
  `infra` only creates the empty engine, the user and the volume.
- **Discarded alternatives:** SQL scripts in `infra/db/init/` (second source of truth); ORM in "create tables on startup"
  mode (leaves no history or rollback).
- **Consequences:** a Postgres extension or a new index is created in the **same** migration that uses it. A hand-changed schema
  in the engine is a defect, not a variant.
- **Verification:** `infra` contains no `CREATE TABLE`; the `data-model.md` §10 queries match the migrations.
