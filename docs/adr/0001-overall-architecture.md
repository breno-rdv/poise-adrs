# 0001. Overall architecture for Poise Platfom

Date: 2026-01-14

## Status

Proposed

## Context
Before start designing the architecture, we should have an estimation of load that our system will have to deal with, for that some benchmark have to be taken and come up with a magic number.

Our system is supposed to have these amount of user and data:
Dealers - 10,000
Customers - 5,000,000
Cars - 500,000
Cars per Dealer - 50 average

### Requests per second (RPS) calculation

DAU - Daily active users (varies based on industry)

> For a B2C SaaS platform
> Dealers - 50% DAU
> Customers - 25% DAU

**Each session will generate 30 requests (search, book, change)**

```
5,000,000 customers * 0,25 = 1,250,000 daily active users
10,000 dealers * 0.5 = 5,000 daily active users

1,255,000 * 30 = 37,500,000 requests per day

RPS = 37,500,000 / 84,000 (seconds/day) = ~450

Supposing high traffic in weekends / holidays
6x more = 2,700 requests per second
```

At a higher traffic the system will have to deal with 2,700 tps.

## Decision

[Describe the decision that was made. Be clear and concise about what will be done.]

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
