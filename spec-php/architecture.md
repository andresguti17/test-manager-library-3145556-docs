# Architecture — Manager Library (PHP)

Hexagonal architecture (ports and adapters). The dependency rules are Articles I to III of [`constitution.md`](constitution.md).

## 1. Layers and dependencies

```mermaid
flowchart LR
    subgraph Input
      HTTP["adapters/inbound/http<br/>(Laravel)"]
    end
    subgraph Core
      APP["application<br/>use cases + ports"]
      DOM["domain<br/>aggregates and value objects"]
    end
    subgraph Output
      DB["adapters/outbound/persistence<br/>(Eloquent + Laravel migrations)"]
      STO["adapters/outbound/storage<br/>covers"]
      SEC["adapters/outbound/security<br/>hash and token"]
    end
    HTTP --> APP --> DOM
    DB -. implements ports .-> APP
    STO -. implements .-> APP
    SEC -. implements .-> APP
```

Dependency arrows point **inward**. `bootstrap` is the only one that sees both halves.

## 2. Folder structure

```text
app/Library/
├── Domain/              # Book, Loan, LoanItem, Member, User; Isbn, Quantity, DateRange
├── Application/
│   ├── Ports/Outbound/  # BookRepository, LoanRepository, MemberRepository, UserRepository,
│   │                    # ReportReader, Clock, PasswordHasher, TokenIssuer, CoverStorage
│   └── Services/        # LendBooksService, ReturnItemService, CreateBookService, SearchBooksService,
│                        # OverdueService, MostBorrowedReportService, LoginService, CreateLibrarianService
├── Adapters/
│   ├── Inbound/Http/    # Controllers, Form Requests, Resources, error handling
│   └── Outbound/        # Persistence/, Storage/, Security/
└── Bootstrap/Composition/   # app/Library/Bootstrap/Composition/PortBindings.php — single composition point
```

## 3. Aggregates

| Root | Contains | Invariants it defends |
|---|---|---|
| `Book` | `Isbn`, total and available copies, soft delete | available between 0 and total; total ≥ 1; non-empty title |
| `Loan` | collection of `LoanItem` | 1 to 3 lines; no repeated book; 14-day due date; immutable except for returns |
| `Member` | document | mandatory and unique document |
| `User` | role, hash | normalized name; role in (`admin`, `librarian`) |

`Loan.AddItem` calls `Book.Withdraw` before adding the line: deducting the copy and creating the line are **a single operation**.
`Loan.ReturnItem` calls `Book.Restock` only once per line.

## 4. Flow: registering a loan

```mermaid
sequenceDiagram
    participant C as Client
    participant H as HTTP
    participant S as LendBooksService
    participant M as MemberRepository
    participant B as BookRepository
    participant L as LoanRepository
    C->>H: POST /api/v1/loans
    H->>S: Lend(memberId, bookIds)
    S->>M: get member + open lines
    S->>S: any overdue? over 3?
    S->>B: get books in batch (Q3)
    S->>S: Loan.AddItem → Book.Withdraw (per book)
    S->>L: save loan + books (one transaction)
    L-->>S: ok or version conflict
    S-->>H: loan or BOOK_NOT_AVAILABLE
    H-->>C: 201 or 409
```

## 5. Errors

The domain and the application throw their own errors (`BookNotAvailable`, `MemberHasOverdue`, `MemberLimitExceeded`, …). **Only** the HTTP adapter
translates them into the single body and status code of [`api-contract.md`](api-contract.md). A framework error never reaches the client.

## 6. What there is not

No events, no queues, no cache, no microservices, no multi-tenancy. They are added when a requirement asks for them, not before.
