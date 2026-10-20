# <NAME> — Tasks

Design: [DESIGN.md](./DESIGN.md)

## Analysis

Build: `<exact build command>` — verified green
Test: `<exact test command>` — verified green
Lint: `<exact lint command>` — verified green

### Known-failing tests
| Test | Reason | Action |
|---|---|---|
| (none — or list pre-existing failures) | | ignore / skip |

### Domain model

```mermaid
classDiagram
    class Order {
        +OrderId id
        +CustomerId customer
        +List~LineItem~ items
        +Money total()
    }
    class LineItem {
        +ProductId product
        +Quantity qty
        +Money unit_price
    }
    Order "1" *-- "*" LineItem

    class OrderRepository {
        <<trait>>
        +save(Order) Result
        +find_by_id(OrderId) Option~Order~
    }

    class CreateOrder {
        <<fn>>
        +CreateOrderCmd → Result~Order, DomainError~
    }
```

### Requirement traceability

Stability marks the boundary: `internal` = agent may rename, split, or extend freely;
`published` = external consumers exist, any change is outside the constitution
(severity 3). Versioning strategy for published contracts lives in an ADR.

| Type / Trait / Fn | Stability | Addresses | Notes |
|---|---|---|---|
| `Order` | internal | [FR1](./DESIGN.md#fr1) | Aggregate root |
| `LineItem` | internal | [FR1](./DESIGN.md#fr1) | Value object, immutable |
| `OrderRepository` | internal | [NFR1](./DESIGN.md#nfr1) | Trait — infra implements |
| `CreateOrder` | published | [FR2](./DESIGN.md#fr2) | REST endpoint — additive changes only |

### Transformations

Include error-path rows derived from the DESIGN.md failure modes table, not only
the happy path. Published functions state their compatibility rule.

| Function | Input → Output | Invariant / Rule |
|---|---|---|
| `CreateOrder` | `CreateOrderCmd → Result<Order, DomainError>` | At least one line item, total > 0 |
| `CreateOrder` (repo down) | `CreateOrderCmd → Err(Unavailable)` | Retry ×3 then fail — from DESIGN.md failure modes |
| `Order::total` | `&self → Money` | Sum of (qty × unit_price) per line item |

## Tasks

### 1. Walking skeleton: POST /orders creates an order with one line ([FR1](./DESIGN.md#fr1), [NFR2](./DESIGN.md#nfr2))
**Goal**: The thinnest path through every layer runs and is tested before any other behaviour.
**Slice**: create an order with one line, end to end (HTTP, domain, storage, test)
**Fit**: closed
**Types**: `Order`, `LineItem` — see domain model
**Constraints**: rules the implementation must respect
- [ADR: decision-name](./adrs/decision-name.md) — the constraint from this decision
- Invariant: `Order` must always have at least one `LineItem`
- Transformation: `CreateOrderCmd → Result<Order, DomainError>` must enforce total > 0
**Tests**: what to test before implementing (red → green), per [`tdd`](../../tdd/SKILL.md)
- `test_post_orders_with_one_line_returns_201_and_persists` — POST then GET returns the order with its line
- `test_order_total_of_one_line_equals_qty_times_unit_price` — unit test behind the slice
**Verify**: `cargo test -- test_post_orders test_order_total && cargo clippy`
**Acceptance criteria**:
- [ ] Criterion 1 (pass/fail, no subjective language)
- [ ] Criterion 2
**Depends on**: (none)
**Time-box**: ~45 min

### 2. POST /orders rejects an empty order ([FR1](./DESIGN.md#fr1))
**Goal**: An order with no line is refused at the endpoint, with a message naming the invariant.
**Slice**: reject an empty order, end to end (HTTP, domain, test)
**Fit**: closed
**Types**: `Order` — see domain model
**Constraints**:
- Invariant: `Order` must always have at least one `LineItem`
**Tests**:
- `test_post_orders_without_lines_returns_422` — response names the invariant
**Verify**: `cargo test -- test_post_orders && cargo clippy`
**Acceptance criteria**:
- [ ] POST /orders with an empty `items` list returns 422
- [ ] Nothing is persisted after the rejected request
**Depends on**: task 1
**Time-box**: ~30 min

## Sessions

Group tasks into autonomous sessions. Each session is a contiguous block of work (target: 2–4H) that ends with a verifiable checkpoint. An agent completes one session, verifies, then proceeds to the next. Minimize the number of sessions — fewer, longer sessions mean fewer interruptions.

### Session 1 — <title> (~2.5H)
Tasks: 1, 2, 3, 4, 5
**Skills**: `software-engineer` (+ language-specific extension for the project)
**Checkpoint**: `<exact command that proves session is complete>`
**Commit point**: yes — commit after checkpoint passes

### Session 2 — <title> (~2H)
Tasks: 6, 7, 8
**Skills**: `software-engineer`
**Checkpoint**: `<exact command>`
**Commit point**: yes

## Quality gates (post-session review)
- [ ] Acceptance criteria: all green above
- [ ] Code review: implementation matches [DESIGN.md](./DESIGN.md) intent
- [ ] Code organization: file placement, module structure, naming conventions (refactoring pass)
- [ ] Code quality: no new complexity, clean types, no duplication
- [ ] Security review: OWASP check, dependency audit, no secrets exposed
- [ ] Observability: relevant metrics identified, dashboards/alerts in place, logging covers key paths
- [ ] Performance: NFR targets met, no regressions on critical paths, load tested if applicable
