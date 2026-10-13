# ADR-004 · The report is computed in the engine and groups by frozen values

- **Status:** accepted
- **Context:** the most-borrowed report (HU-10) joins `loan` and `loan_item` over a range. If it read the title or the category from the live
  catalog, renaming a book or a category would change a report for an already closed period.
- **Decision:** (1) the report **is not persisted**: it is computed with a single aggregate query through a read port. (2) It groups by
  `book_id, book_title, category_name` **as frozen on the line**. (3) `loan_item.category_name` deliberately has **no FK**.
  (4) If a book changed category within the range, the report shows **two rows**: it does not pick a winner.
- **Discarded alternatives:** joining with the live catalog (the report stops being stable); taking "the most recent value in the range"
  (a new loan would change what was already read); materializing a report table (second source of truth).
- **Consequences:** CA-10.1 says "one row per book and frozen category name", not "one row per book". The report
  includes neither members nor librarians (privacy).
- **Verification:** lend a book, rename it, request the report for the same range: the report title is the old one.
