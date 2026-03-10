# 0002. Domain-Driven Design and Service Boundaries

Date: 2026-01-28

## Status

[Proposed]

## Context

Following the decision to adopt microservices architecture (ADR 0001), we need to establish clear service boundaries aligned with business domains rather than technical layers. The Poise platform has distinct business capabilities that vary in complexity, data requirements, and scaling patterns.

**Identify core functionalities, personas and actions performed by them.**

![core-personas-actions](../../images/core-personas-actions.png)

## Decision

After having the core functionalities, personas and actions identified, we will adopt **Domain-Driven Design** principles to define service boundaries and establish the following **Bounded Contexts**:

![domain-services](../../images/domain-services.png)

### Services and responsabilities
- Advertising Service (AI pipeline): Extracts audio and images from video recording, and provides vehicle details and metadata.

## Consequences

### Positive

- [List positive outcomes]

### Negative

- [List negative outcomes or trade-offs]

### Neutral

- [List neutral consequences or considerations]

## Alternatives Considered

### [Alternative 1]

[Brief description and why it was not chosen]

### [Alternative 2]

[Brief description and why it was not chosen]

## References

- [Link to relevant documents, discussions, or resources]
