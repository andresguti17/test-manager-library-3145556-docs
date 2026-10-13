# Specification — Manager Library

**What is built and why.** A system for a small library to keep its book catalog,
its members and its loans, and to see which books are borrowed most.

Governed by [`constitution.md`](constitution.md). The data model lives only in [`data-model.md`](data-model.md)
and the HTTP contract only in [`api-contract.md`](api-contract.md).

---

## 1. Actors

| Actor | What it does | How it authenticates |
|---|---|---|
| **Administrator** (`admin`) | Maintains the book catalog and the members; registers librarians; sees all reports | Username and password → token |
| **Librarian** (`librarian`) | Registers loans and returns; consults catalog, members and overdue | Username and password → token |
| **Member** | Person who borrows books. **Does not enter the system**: it is data, not a user | — |

The `admin` role is provisioned by the deployment from environment variables. **Nobody grants it at runtime.**

## 2. Scope

**Inside:** book catalog with optional cover, five fixed categories, members, loans with several
lines, return per line, overdue books, most-borrowed report, login with roles.

**Outside, and not reopened:** fines and payments, holds, renewals, email or SMS, bulk import,
inter-branch loans, book attributes beyond §3 (no publisher, no year, no language).

## 3. User stories and acceptance criteria

### HU-01 · Log in
As a user I want to authenticate in order to operate according to my role.
- **CA-01.1** With valid credentials a token valid for 8 hours is received.
- **CA-01.2** With invalid credentials it responds 401 with a message that does **not** say whether the username or the password failed.
- **CA-01.3** Every endpoint except login and `/health` requires a token. Without a token: 401. Insufficient role: 403.

### HU-02 · Register librarians (admin)
- **CA-02.1** The admin creates a user with a name, password and the `librarian` role.
- **CA-02.2** The username is stored lowercased and trimmed; it is unique. Duplicate: 409.
- **CA-02.3** Requesting the `admin` role through the API responds 422. The password is never returned nor written to logs.

### HU-03 · Manage books (admin)
- **CA-03.1** Create a book with title, author, ISBN, category and total copies. Available copies start equal to the total.
- **CA-03.2** The ISBN has 13 digits and is unique among active books. Duplicate: 409.
- **CA-03.3** Total copies ≥ 1. Changing the total adjusts available copies by the same difference and **never** leaves available < 0 (422).
- **CA-03.4** Uploading a cover (JPG/PNG, max. 2 MB) stores an **opaque key**; without a cover the field is `null`, never an empty string.
- **CA-03.5** Deleting a book is a **soft delete**: it stops appearing in the catalog but remains in past loans. A book with open loans is not deactivated (409).

### HU-04 · Search books
- **CA-04.1** Search by partial text in title or author, filter by category, only active books, order by title, paginated (`page`, `size`, max. 50).
- **CA-04.2** The response includes the total number of results.
- **CA-04.3** Getting a book by a nonexistent or deactivated id: 404.

### HU-05 · Manage members
- **CA-05.1** The admin and the librarian create members with a name and a document. The document is unique: duplicate 409.
- **CA-05.2** Members are listed and searched by name or document, paginated.
- **CA-05.3** Members are not deleted.

### HU-06 · Register a loan
- **CA-06.1** A loan is for **one** member, it is registered by the authenticated user and carries 1 to 3 lines.
- **CA-06.2** The same book is not repeated within a loan (422).
- **CA-06.3** Each line deducts **1** available copy in the same operation. Without copies: 409 and **nothing is registered**.
- **CA-06.4** A member cannot have more than **3 unreturned books** in total (422 with the current count).
- **CA-06.5** A member with some **overdue** book cannot borrow another (409).
- **CA-06.6** The due date is `loaned_at + 14 days`. The line freezes the title, ISBN and category name **at that instant**.
- **CA-06.7** A registered loan is neither edited nor deleted.

### HU-07 · Register a return
- **CA-07.1** **One line** is returned at a time. It sets `returned_at` and adds 1 to available copies.
- **CA-07.2** Returning an already-returned line: 409. It **never** adds twice.
- **CA-07.3** Available copies never exceed the total (the database guarantees it).
- **CA-07.4** When all lines are returned, the loan counts as **closed**.

### HU-08 · View a loan and list loans
- **CA-08.1** Getting a loan brings its lines and its computed status (`open`, `overdue`, `closed`).
- **CA-08.2** List by date range, status and member, order by date descending, paginated.
- **CA-08.3** The status **is computed**, not stored.

### HU-09 · Overdue books
- **CA-09.1** Lists unreturned lines with `due_at` before now: book, member, days late.
- **CA-09.2** Ordered by most days late first. Only `admin` and `librarian`.

### HU-10 · Most-borrowed books report
- **CA-10.1** Given a date range, one row per **book and frozen category name**, with the number of times borrowed, ordered highest to lowest.
- **CA-10.2** End before start: 422. Range without loans: empty list, not an error.
- **CA-10.3** The report is **not** broken down by librarian or by member.
- **CA-10.4** A report for a closed period **does not change** even if the book or its category is later renamed.
- **CA-10.5** Only `admin`.

## 4. Business rules

| # | Rule |
|---|---|
| RN-1 | Available copies between 0 and total, always. |
| RN-2 | Loan term: 14 days. Maximum 3 unreturned books per member. |
| RN-3 | A member with overdue books borrows no more until returning them. |
| RN-4 | What is frozen in the line (title, ISBN, category) does not follow the catalog. |
| RN-5 | Loans and returns are not deleted. Books only have a soft delete. |
| RN-6 | Dates are UTC instants. User-facing messages in Spanish; code and columns in English. |

## 5. Non-functional requirements

| # | Requirement | How it is measured |
|---|---|---|
| RNF-1 | Book search responds in under 300 ms with 5 000 books | Load test with the `tool` seeder |
| RNF-2 | Two librarians lending the **last copy** at once: one wins, the other receives 409 | Concurrent test |
| RNF-3 | The password never travels nor is stored in plain text; it never appears in logs | Log and response review |
| RNF-4 | Every HTTP error uses the same body (see contract) | Contract test |
| RNF-5 | The system starts with a single command and the database starts empty | `infra` `README` |
