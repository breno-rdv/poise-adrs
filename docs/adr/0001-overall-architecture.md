# 0001. Overall architecture for Poise Platfom

Date: 2026-01-14

## Status

Proposed

## Context
Before initiating the architectural design process, it is essential to determine the anticipated workload for our system. To do so, we will need to conduct benchmark tests and derive a precise figure.

Our system is designed to accommodate a specific number of users and volume of data;
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

## Functional Requirements

- Dealers can add cars for sale only recording a video describing its features, uploading it to its whatsapp business account
- Dealers can manage (add/modify/delete) a car
- Dealers can rent a car for a desired periord
- Customers can rent a car
- Customers can schedule a visit for the dealer to come by and showcase the car at their location
- Customer can give offers to a car that is on sale

## Non-Functional Requirements

### 1.1 Availability

- Guarantee **≥ 99.9%** monthly availability
- Automatic failover between zones/regions
- Zero downtime for critical operations

### 1.2 Latency

- Read operations with:
- **p95 ≤ 150 ms**

- **p99 ≤ 250 ms**

- Low global latency:
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
Taking into account all the functional and non-functional requirements, microservices architecture will be used for this solution, as some functionalities will handle higher workloads than others, moreover some services will require specific lnaguage to be more performant.

## Consequences

### Positive
- It service can scale independently
- Independent deploys for each service
- Reduced Blast radius
- Independent technology per service requirement

### Negative

- Operational overhead
- Async and sync communication management
- Manage failures
- Troubleshooting
- Quality assurance
- Automated tests

### Neutral

- Team size

## Alternatives Considered

### Monolith

The monolith approach can increase costs, and also become a big ball of mud, as the system will have many features.

### Modular Monolith featuring Clean Architecture

There will be some services that require some specific technologies to be more performant, as well as some feature will have less complexity when using a specific database.

## References

- https://fidelissauro.dev/monolitos-microservicos/
