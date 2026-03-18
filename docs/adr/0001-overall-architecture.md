# 0001. Overall Architecture for Poise Platform

Date: 2026-01-14

## Status

Approved

## Context

The Poise platform serves 10,000 dealers and 5,000,000 customers across rental and vehicle sales workflows. Based on DAU projections and session request modelling, the system must handle a baseline of ~434 RPS and a peak of ~**2,600 RPS** (6× weekend/holiday multiplier). Core requirements include ≥99.9% availability, p95 read latency ≤150 ms, 10× horizontal scalability, and full encryption in transit and at rest.

See [System Requirements](../system-requirements.md) for the full scale model, functional requirements, and non-functional requirements.

## Decision

We have chosen **microservices architecture** for this solution based on the following rationale:

- Different services handle significantly varied workloads, requiring independent scaling
- Specific services benefit from language/technology optimization for performance
- Functional complexity warrants service decomposition
- Independent deployment and operational requirements across services

## Consequences

### Positive
- Services scale independently based on demand
- Independent deployment pipelines per service
- Reduced blast radius for failures
- Technology flexibility—each service can use optimal language/framework

### Negative

- Increased operational complexity and infrastructure overhead
- Complex inter-service communication patterns (async/sync)
- Distributed system failure scenarios and handling
- End-to-end troubleshooting across service boundaries
- Elevated QA complexity and test coverage requirements
- Additional effort for automated and integration testing

### Neutral

- Team size and structure (can adapt to microservices with appropriate organization)

## Alternatives Considered

### Monolith

**Rejected.** While simpler initially, a monolithic approach would become difficult to maintain as the system grows. The varied workload requirements and performance needs across different features (dealer operations vs. customer search) would create resource contention and make targeted scaling impossible. Long-term cost would increase significantly.

### Modular Monolith with Clean Architecture

**Rejected.** Although cleaner than a traditional monolith, this approach would not allow independent technology choices where performance is critical (e.g., specific languages or databases for particular services). Additionally, deployment remains coupled, limiting our ability to iterate on individual features independently.

## References

- [Monolith vs Microservices](https://fidelissauro.dev/monolitos-microservicos/)
