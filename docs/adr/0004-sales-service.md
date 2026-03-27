# 0004. Sales Service Integration and Responsibilities

Date: 2026-03-18

## Status

Proposed

## Context

As defined in ADR 0002 (Domain-Driven Design Service Boundaries) and ADR 0003 (AI Pipeline), the Sales Service owns the purchase lifecycle. To implement it correctly we need to establish its internal design: aggregate state machine, saga coordination, integration patterns, and read model strategy.

The primary challenges this ADR addresses are:

- **Concurrent scheduling conflicts**: Multiple customers may attempt to schedule a visit for the same vehicle simultaneously, and a vehicle could be marked as sold while a visit is pending.
- **Dealer confirmation dependency**: The visit workflow cannot complete without explicit dealer confirmation, introducing an external and potentially delayed step that must be handled gracefully (timeouts, reminders, cancellations).
- **Eventual consistency with Inventory**: The Sales Service depends on vehicle availability state owned by the Inventory Service, requiring a clear integration contract and conflict resolution strategy.
- **Read model for UX**: The frontend requires low-latency access to visit and vehicle state that cannot be served efficiently from the transactional write model alone.

The Sales Service must coordinate with the Inventory Service (to request and confirm holds on vehicles), the Customer Service (to resolve customer identity), the Notification Service (to alert dealers and customers), and the BFF layer (which serves the frontend over REST).

## Decision

We will implement the Sales Service using **CQRS** and an **event-driven saga** pattern.

### Visit Aggregate State Machine

The `Visit` aggregate is the core write model of the Sales Service and transitions through the following states:

```
DRAFT → HOLD_PENDING → TENTATIVE → CONFIRMED → COMPLETED
                    ↘           ↘            ↘
                  CANCELED    CANCELED     CANCELED
                                         EXPIRED (deadline exceeded)
```

- **DRAFT**: Customer has initiated a visit request; no inventory hold placed yet.
- **HOLD_PENDING**: Sales Service has published a hold command to Inventory; awaiting acknowledgement.
- **TENTATIVE**: Inventory hold confirmed; awaiting dealer confirmation.
- **CONFIRMED**: Dealer has confirmed the visit; the appointment is locked in.
- **COMPLETED**: Visit occurred and the sale workflow concluded.
- **CANCELED**: Either party canceled, or a compensating action was triggered by the saga.
- **EXPIRED**: Dealer did not confirm within the deadline; hold is released automatically.

### Saga: Visit Booking Orchestration

The Sales Service orchestrates a saga to coordinate the multi-step booking workflow:

1. Customer submits a visit schedule request → Visit created in **DRAFT**.
2. Sales Service publishes a `HoldVehicleCommand` to the `sales.commands` Kafka topic → Visit transitions to **HOLD_PENDING**.
3. Inventory Service processes the hold and publishes a `VehicleHoldConfirmed` or `VehicleHoldFailed` event to `inventory.events`.
4. On `VehicleHoldConfirmed` → Visit transitions to **TENTATIVE**; dealer notification is triggered and a deadline timer is started (default: 48 h, configurable per region).
5. On `VehicleHoldFailed` (e.g., vehicle already sold or held by another saga) → Visit transitions to **CANCELED**; customer is notified.
6. Dealer confirms within the deadline → Visit transitions to **CONFIRMED**; customer notification sent.
7. If the dealer does not respond before the deadline → saga timeout fires → compensating `ReleaseHoldCommand` published → Visit transitions to **EXPIRED**.
8. Customer or dealer cancels at any point → compensating `ReleaseHoldCommand` published → Visit transitions to **CANCELED**.

Idempotency keys are applied on all commands to handle retries and at-least-once delivery safely.

### Architecture Flow

```
┌──────────────┐    ┌──────────────┐       ┌──────────────────────┐
│   Frontend   │───▶│     BFF      │─gRPC─▶│   Sales Service      │
│  (REST/JSON) │    │  (REST/gRPC) │       │   (Write Model)      │
└──────────────┘    └──────────────┘       └──────────┬───────────┘
       ▲                                               │ publishes commands
       │                                               ▼
       │                                   ┌───────────────────────┐
       │                                   │  Kafka: sales.commands│
       │                                   └──────────┬────────────┘
       │                                              │ consumed by
       │                                              ▼
       │                                   ┌───────────────────────┐
       │                                   │   Inventory Service   │
       │                                   └──────────┬────────────┘
       │                                              │ publishes events
       │                                              ▼
       │                                   ┌───────────────────────┐
       │  Sales Read Model ◀───────────────│ Kafka: inventory.events│──▶ Notification
       │  updated by consumer              └───────────────────────┘      Service
       │
       └──────── UX reads from Sales Read Model (low-latency)
```

### Key Design Decisions

1. **gRPC with BFF**: The Sales Service exposes a gRPC interface consumed by the BFF layer, which translates to REST/JSON for the frontend. This separates internal service communication from the public API contract.
2. **Asynchronous commands via Kafka**: Commands to the Inventory Service are published to `sales.commands` rather than issued synchronously, decoupling availability and enabling retries and full auditability.
3. **Event-driven read model**: The Sales Read Model is updated by consuming `inventory.events`, keeping the read path eventually consistent without impacting write-path performance.
4. **Saga with compensating transactions**: All multi-step workflows have explicit compensation logic (hold release) to handle failures, timeouts, and cancellations without leaving the system in an inconsistent state.
5. **Deadline enforcement**: A timer is started when a visit enters **TENTATIVE**. If the dealer does not confirm before the configured deadline, the saga automatically compensates and transitions the visit to **EXPIRED**.

### Domain Events Emitted

| Event | Published When |
|---|---|
| `VisitScheduled` | Customer submits a visit request |
| `VisitHoldRequested` | Sales Service requests an inventory hold |
| `VisitConfirmed` | Dealer confirms the visit |
| `VisitCanceled` | Customer or dealer cancels |
| `VisitExpired` | Dealer confirmation deadline exceeded |
| `VisitCompleted` | Visit occurred and workflow concluded |

### Customer-Facing API (via BFF)

| Operation | Description |
|---|---|
| `ScheduleVisit(customerId, vehicleId, proposedTime)` | Creates a Visit in DRAFT and initiates the saga |
| `ConfirmVisit(visitId, dealerId)` | Dealer confirms; transitions to CONFIRMED |
| `CancelVisit(visitId, actorId, reason)` | Triggers compensating saga; transitions to CANCELED |
| `GetVisit(visitId)` | Reads from the read model |

## Consequences

### Positive

- **Scalability**: Decoupling through Kafka enables independent scaling of Sales and Inventory services.
- **Auditability**: All commands and state transitions are persisted, providing a complete audit trail.
- **Resilience**: The saga with compensating transactions ensures the system recovers gracefully from partial failures and timeouts.
- **Conflict resolution**: Concurrent scheduling conflicts are resolved deterministically at the Inventory hold level; `VehicleHoldFailed` propagates back to cancel competing visits.
- **Performance**: The read model serves the UX without impacting transactional write-path throughput.

### Negative

- **Complexity**: CQRS and the saga pattern add architectural complexity compared to a simple CRUD model.
- **Eventual consistency**: The read model may briefly lag behind the write model; the UX must handle optimistic or stale states gracefully.
- **Operational overhead**: Kafka, distributed tracing, and saga state storage increase infrastructure and on-call burden.
- **Debugging**: Distributed asynchronous flows require centralized tracing with correlation IDs propagated across Kafka messages to diagnose issues effectively.

### Neutral

- **Kafka dependency**: Kafka becomes critical shared infrastructure; its availability directly affects the Sales Service.
- **Team learning curve**: Engineers need fluency in CQRS, event-driven sagas, and Kafka consumer group management.

## Alternatives Considered

### Alternative 1: Direct Synchronous Service-to-Service Communication

**Description**: Sales Service calls Inventory Service synchronously via REST/gRPC without message queues.

**Why Not Chosen**:
- Creates tight coupling between services.
- Cascading failures: if Inventory is unavailable, no visit can be scheduled.
- No built-in audit trail of commands.
- Concurrent hold requests require distributed locking, which is harder to reason about than a saga with compensation.
- Does not support the asynchronous processing model defined in ADR 0003.

### Alternative 2: Event Sourcing at Database Level Only

**Description**: Use a database-level event store (e.g., EventStoreDB) instead of Kafka for command and event processing.

**Why Not Chosen**:
- Limits cross-service event distribution.
- Makes integration with other bounded contexts and external systems more difficult.
- Less suitable for real-time stream processing needed by notifications and read model projections.
- Kafka is already established infrastructure per ADR 0001/0002.

### Alternative 3: Synchronous Read/Write with Shared Database

**Description**: Use a shared database for both Sales and Inventory writes, with synchronous transactions to handle holds.

**Why Not Chosen**:
- Creates tight coupling at the data layer, violating the data ownership principle from ADR 0002.
- Poor scalability characteristics for a high-concurrency scheduling scenario.
- Eliminates the ability to scale the services independently.

## References

- [0002-domain-driven-design-service-boundaries.md](0002-domain-driven-design-service-boundaries.md)
- [0003-ai-pipeline.md](0003-ai-pipeline.md)
- CQRS Pattern: https://martinfowler.com/bliki/CQRS.html
- Event Sourcing: https://martinfowler.com/eaaDev/EventSourcing.html
- Saga Pattern: https://microservices.io/patterns/data/saga.html
