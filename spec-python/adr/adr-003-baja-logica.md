# ADR-003 · Books are soft-deleted, not erased

- **Status:** accepted
- **Context:** a loan item points to a book. If the book is physically deleted, the history and the report are left broken.
- **Decision:** the book carries `deleted_at`. Deleting fills it; the catalog filters by `deleted_at IS NULL` in a single place
  (global filter). A book with open loans is **not** deleted (409 `BOOK_HAS_OPEN_LOANS`). ISBN uniqueness is a
  **partial** index over active books, so a deleted ISBN can be registered again.
- **Discarded alternatives:** physical delete with cascade (destroys the history); `active = false` flag (does not say when).
- **Consequences:** `loan_item.book_id` carries a `RESTRICT` FK as a **last-resort barrier**: a manual `DELETE` or a future code change
  fails loudly instead of corrupting the history. The cover binary is the only data that is deleted physically.
- **Verification:** requesting a deleted book returns 404; a `psql` `DELETE` over a book with items fails on the FK.
