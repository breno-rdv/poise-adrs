# 0004. Sales Service Integration and Responsibilities

Date: 2026-03-18

## Status

Proposed

## Context

As defined in ADR 0002 (Domain-Driven Design Service Boundaries) and ADR 0003 (AI Pipeline), we need to establish a clear architecture for the Sales Service, including its responsibilities, domain events, and integration patterns. The Sales Service must handle customer interactions, coordinate with inventory management, and maintain a read model for the UX layer.

Sales service must handle concurrent schedules, schedule blockes (car could be set to sold at the same time), and more important the flow rely on confirmation from dealer for schedule to be finished.

Once again, as ADR 0002 states, **Sales Service** owns the purchase lifecycle, including buying intent, visit scheduling support, and sale-oriented workflow steps that depend on inventory and customer information.

## Responsibilities

The Sales Service is responsible for:

1. **Customer Request Handling**: Process customer queries and sales requests through gRPC interface with a Backend-for-Frontend (BFF)
2. **Inventory Coordination**: Send commands to the Inventory Service via Kafka for stock verification and updates
3. **Command Publishing**: Publish domain commands to Kafka for asynchronous processing and audit trail
4. **Read Model Maintenance**: Maintain a Sales Read Model that is updated via Kafka Events published by the Inventory Service
5. **UX Layer Support**: Provide a consistent view of the sales state to the user interface

## Decision

TBD

### Architecture Flow

```
┌──────────────┐    ┌──────────────┐       ┌──────────────┐        ┌────────────────┐
│   Frontend   │───▶│     BFF      │──────▶│    SALES     │────────▶│ Kafka: Commands│
│  (REST API)  │    │   (gRPC)     │       └──────┬───────┘        └─▲──────────────┘
└──────────────┘    └──────────────┘              │                    │
      ▲                                            ▼                    │
      │                                 ┌───────────────────┐            │
      │                                 │   Inventory       │◀───────────┘
      │                                 └─────────┬────────┘
      │                                           ▼
      │                                  ┌─────────────────┐
      │                                  │ Kafka: Events   │───▶ Notification
      │                                  └─────────────────┘
      │
      └────────────────── UX updated via Sales Read Model
```

### Key Design Decisions

1. **gRPC Communication with BFF**: The Sales Service exposes a gRPC interface for the Backend-for-Frontend layer, which provides REST endpoints to frontend clients
2. **Asynchronous Command Processing**: Commands are published to Kafka for reliable, auditable interactions with the Inventory Service
3. **Event-Driven Read Model**: The Sales Read Model is updated by consuming Kafka Events from the Inventory Service, ensuring eventual consistency
4. **Separation of Concerns**: Write operations are handled through the command stream, while read operations are served from the optimized read model

## Consequences

### Positive

- **Scalability**: Decoupling through Kafka enables independent scaling of Sales and Inventory services
- **Auditability**: All commands are persisted in Kafka, providing a complete audit trail of business operations
- **Resilience**: Asynchronous processing reduces coupling and improves system resilience to failures
- **Performance**: The read model allows the UX to query optimized data structures without impacting transactional operations
- **Event Sourcing**: Kafka serves as the single source of truth for domain events across services

### Negative

- **Complexity**: CQRS adds architectural complexity compared to traditional CRUD models
- **Eventual Consistency**: The read model may lag behind write operations, requiring UX to handle eventual consistency
- **Operational Overhead**: Kafka increases infrastructure complexity and requires additional monitoring and maintenance
- **Debugging**: Distributed asynchronous operations make debugging and tracing more challenging

### Neutral

- **Technology Lock-in**: Kafka becomes a critical infrastructure dependency
- **Learning Curve**: Team members need to understand CQRS and event-driven patterns

## Alternatives Considered

### Alternative 1: Direct Synchronous Service-to-Service Communication

**Description**: Sales Service calls Inventory Service synchronously via REST/gRPC without message queues.

**Why Not Chosen**: 
- Creates tight coupling between services
- Reduces resilience (cascading failures)
- No audit trail of operations
- Harder to scale independently
- Does not support the AI pipeline's asynchronous processing model defined in ADR 0003
- Violates separation of concerns between synchronous API layer and asynchronous domain operations

### Alternative 2: Event Sourcing at Database Level Only

**Description**: Use a database-level event store instead of Kafka for command and event processing.

**Why Not Chosen**:
- Limits cross-service event distribution
- Makes integration with external systems difficult
- Less suitable for real-time stream processing needed by notifications and read models
- Kafka provides better infrastructure for polyglot persistence and service interaction

### Alternative 3: Synchronous Read/Write with Shared Database

**Description**: Use a shared database for both writes and reads with synchronous transactions.

**Why Not Chosen**:
- Creates tight coupling at the data layer
- Poor scalability characteristics
- No separation of transactional and analytical concerns
- Conflicts with microservices principles defined in ADR 0002

## References

- [0002-domain-driven-design-service-boundaries.md](0002-domain-driven-design-service-boundaries.md)
- [0003-ai-pipeline.md](0003-ai-pipeline.md)
- CQRS Pattern: https://martinfowler.com/bliki/CQRS.html
- Event Sourcing: https://martinfowler.com/eaaDev/EventSourcing.html
