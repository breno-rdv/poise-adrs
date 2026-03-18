# Poise Platform — System Requirements

This document captures the scale model, functional requirements, and non-functional requirements for the Poise platform. It serves as the problem statement that informs the architectural decisions recorded in the ADR suite.

---

## System Scale

| Entity | Volume |
|---|---|
| Dealers | 10,000 |
| Customers | 5,000,000 |
| Cars | 500,000 |
| Cars per Dealer (avg) | 50 |

## Requests per Second (RPS)

> For a B2C SaaS platform:
> - Dealers — 50% DAU
> - Customers — 25% DAU

**Each session generates approximately 30 requests (search, book, change).**

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

---

## Functional Requirements

### Dealer

- Add cars for sale by recording a video describing features and uploading to WhatsApp Business
- Manage cars (add, modify, delete)
- Rent cars for desired periods

### Customer

- Rent cars
- Schedule visits for dealers to showcase cars at their location
- Make offers on cars for sale

---

## Non-Functional Requirements

### Availability

- Guarantee **≥ 99.9%** monthly availability
- Automatic failover between zones/regions
- Zero downtime for critical operations

### Latency

**Read Operations:**
- p95 ≤ **150 ms**
- p99 ≤ **250 ms**

**Global Distribution:**
- Global p95 ≤ **200 ms**

### Scalability

- Automatic horizontal scaling
- Support for **10×** growth without noticeable degradation

### Resilience

- Circuit breaker, jittered retries, timeouts
- MTTR ≤ 5 minutes
- AZ/region fault tolerance

### Global Distribution

- Use of CDN
- Cache hit ratio ≥ **95%**
- Data replication lag ≤ **1 second**

### Security

- 100% HTTPS/TLS
- Encryption of data at rest
- Granular IAM
