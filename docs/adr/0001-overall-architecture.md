# 0001. Overall Architecture for Poise Platform

Date: 2026-01-14

## Status

Approved

## Context

Before initiating the architectural design process, it is essential to determine the anticipated workload for our system. We have defined the following scale metrics:

**System Scale:**
- Dealers: 10,000
- Customers: 5,000,000
- Cars: 500,000
- Cars per Dealer: 50 (average)

### Requests per second (RPS) calculation

DAU - Daily active users (varies based on industry)

> For a B2C SaaS platform
> Dealers - 50% DAU
> Customers - 25% DAU

**Each session will generate 30 requests (search, book, change)**

```
5,000,000 customers × 0.25 = 1,250,000 daily active users
10,000 dealers × 0.50 = 5,000 daily active users

Total DAU: 1,255,000 users
1,255,000 × 30 requests/session = 37,500,000 requests per day

Baseline RPS: 37,500,000 ÷ 86,400 (seconds/day) ≈ 434 RPS

Peak traffic (weekends/holidays with 6× multiplier):
434 × 6 ≈ 2,600 RPS
```

The system must handle peak traffic of approximately **2,600 requests per second**.

## Functional Requirements

**Dealer Functions:**
- Add cars for sale by recording a video describing features and uploading to WhatsApp Business
- Manage cars (add, modify, delete)
- Rent cars for desired periods

**Customer Functions:**
- Rent cars
- Schedule visits for dealers to showcase cars at their location
- Make offers on cars for sale

## Non-Functional Requirements

### 1.1 Availability

- Guarantee **≥ 99.9%** monthly availability
- Automatic failover between zones/regions
- Zero downtime for critical operations

### 1.2 Latency

**Read Operations:**
- p95 ≤ **150 ms**
- p99 ≤ **250 ms**

**Global Distribution:**
- Global p95 ≤ **200 ms**

### 1.3 Scalability

- Automatic horizontal scaling
- Support for **10×** growth without noticeable degradation

### 1.4 Resilience

- Circuit breaker, jittered retries, timeouts
- MTTR ≤ 5 minutes
- AZ/region fault tolerance

### 1.5 Global Distribution

- Use of CDN
- Cache hit ratio ≥ **95%**
- Data replication with lag ≤ **1 second**

### 1.6 Security

- 100% HTTPS/TLS
- Encryption of data at rest
- Granular IAM

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
