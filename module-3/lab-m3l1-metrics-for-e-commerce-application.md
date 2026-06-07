# Lab: Identify and Create Metrics for an E-commerce Application Based on Business Requirements

**Author:** Krzysztof Macherzyński
**Estimated time needed:** 45 minutes

## Scenario

I've joined **"BookNest,"** a growing online bookstore experiencing occasional performance
issues during book launch events and weekend sales. The CTO asked me to establish a
comprehensive monitoring strategy to ensure system reliability and improve customer experience.

BookNest's architecture includes:

- **Frontend**: Web application for customers
- **API layer**: Handles all business logic
- **Database**: Stores customer, order, and inventory data
- **Payment gateway**: Processes transactions

---

## Exercise 1: Understanding the Golden Signals

The Four Golden Signals provide comprehensive visibility into system health: **Latency**
(time to service a request), **Traffic** (demand on the system), **Errors** (rate of failed
requests), and **Saturation** (resource utilization levels).

### Task A: Identify Golden Signals for BookNest

| Signal | Metric name | How to measure | Business impact |
| --- | --- | --- | --- |
| **Latency** | Checkout API response time (P95) | Time in milliseconds (ms) from request received to response sent, measured at the 95th percentile | Slow checkout directly causes cart abandonment and lost revenue, especially during launches |
| **Traffic** | Orders per minute & concurrent active users | Count of completed orders/min and number of active sessions per minute | Quantifies real customer demand; sudden spikes signal launch events that must be scaled for |
| **Errors** | Combined error rate (HTTP 5xx + payment failures) | Percentage of failed requests over total requests, split by technical (500 errors) and business (declined/failed payments) | Failed requests and payment failures equal lost sales and damaged customer trust |
| **Saturation** | Database connection pool utilization | Percentage of active DB connections vs. max pool size; also CPU and memory headroom | When the pool fills first, new requests queue or fail, cascading slow checkouts and timeouts |

---

## Exercise 2: Categorizing Metrics by Domain

### Task A: Infrastructure metrics

| Metric name | Target value | Alert threshold | Why monitor |
| --- | --- | --- | --- |
| Database CPU utilization | <75% average | >90% for 5 min | High CPU causes slow queries, impacting checkout |
| Server memory utilization | <80% average | >92% for 5 min | Memory exhaustion triggers swapping/OOM kills, crashing services |
| Disk I/O wait time | <10 ms average | >50 ms for 5 min | Slow disk I/O bottlenecks database reads/writes and order persistence |
| Database connection pool usage | <70% of max | >90% sustained | Pool exhaustion blocks new queries, freezing checkout and search |
| Database replication lag | <1 second | >10 seconds | Stale replicas serve outdated inventory/order data, causing oversells |
| Network packet loss | <0.1% | >1% for 2 min | Packet loss causes retransmits, latency, and dropped payment-gateway calls |

### Task B: Application performance metrics

| Service | Metric | Target SLO | Alert threshold | Reasoning |
| --- | --- | --- | --- | --- |
| Search API | Response time P95 | <150 ms | >400 ms | Slow search frustrates customers |
| Search API | Error rate | <0.5% | >2% | Failed searches = lost sales |
| Cart API | Response time P95 | <200 ms | >500 ms | Slow cart updates discourage adding items |
| Cart API | Error rate | <0.5% | >2% | Cart errors lose the items a customer intended to buy |
| Checkout API | Response time P95 | <500 ms | >2 s | Checkout is the conversion moment; delays directly abandon orders |
| Checkout API | Error rate | <0.1% | >1% | A failed checkout is a directly lost, ready-to-buy sale |
| Payment API | Response time P95 | <800 ms | >2 s | Payment timeouts abandon transactions and erode trust |
| Payment API | Error rate | <0.1% | >1% | Payment failures lose revenue and may indicate gateway/fraud issues |

> **Note:** Checkout and Payment carry the strictest SLOs (lowest error tolerance) because
> they sit at the revenue-critical end of the funnel.

### Task C: Business metrics

| Business metric | Normal range | Peak range (book launch) | Critical threshold |
| --- | --- | --- | --- |
| Orders per minute | 10–25 | 100–200 | <5 (system issue) |
| Revenue per hour | $2,000–$5,000 | $20,000–$40,000 | <$500 during a launch (likely outage) |
| Cart abandonment rate | 60–70% | 65–75% | >85% (checkout/payment friction) |
| Search-to-purchase conversion rate | 3–5% | 8–12% | <1% (search or relevance broken) |
| Average order value (AOV) | $25–$40 | $30–$50 | <$10 sustained (pricing/promo error) |
| Out-of-stock rate (bestsellers) | <2% | <5% | >15% (inventory/fulfillment failure) |

### Task D: Security metrics

| Security metric | What to monitor | Alert condition | Response action |
| --- | --- | --- | --- |
| Failed login attempts | Login failures from same IP | >10 in 5 minutes | Temporary IP block + CAPTCHA challenge |
| Unusual API access patterns | Request rate per IP/API key vs. baseline | >5x baseline or rate-limit violations | Throttle/rate-limit, flag for review |
| Payment fraud indicators | Repeated declined cards, mismatched billing/IP geolocation | >3 declines per account or velocity spike | Hold transaction, trigger manual fraud review |
| Sensitive data access anomalies | Access to customer PII/order data outside normal role/hours | Privilege escalation or bulk export by a single account | Revoke session, alert security team, audit logs |

---

## Challenge 1: Handling a Holiday Shopping Scenario

**Scenario:** BookNest expects 10x normal traffic during the holidays. The following metrics
need temporary threshold adjustments.

### Holidays Adjustments

| Metric | Normal alert threshold | Holiday threshold | Reasoning |
| --- | --- | --- | --- |
| Database CPU | >90% | >95% | Higher utilization expected; alert only at critical levels to avoid alert fatigue |
| Orders per minute (low alert) | <5 | <30 | Baseline demand is far higher; a drop to 30 now signals a problem |
| Database connection pool | >90% | >95% | Expected sustained high concurrency; reserve alerting for true exhaustion |
| Server memory utilization | >92% | >95% | Temporarily accept higher resource pressure under planned peak load |
| Concurrent active users (capacity) | 1,000 | 10,000 | Scale the capacity alert up to match provisioned holiday infrastructure |

**Note on what should NOT change:** Error-rate and latency SLOs (Checkout >2s, Payment error
>1%) stay the same — customers expect the same experience regardless of traffic, and
degraded quality during peak directly destroys peak revenue.

---

## Challenge 2: Root Cause Analysis Exercise

**Scenario:** The following alerts fire simultaneously at 2:00 PM on a Saturday:

- Checkout API response time: 5 seconds (threshold: >2s)
- Database CPU: 95% (threshold: >90%)
- Orders per minute: 3 (normal: 20–30)
- Error rate on payment API: 8% (threshold: >1%)

### RCA Exercise

| Metric | Observed | Role | Relationship |
| --- | --- | --- | --- |
| Database CPU: 95% | Above threshold | **Likely root cause** | Saturated DB slows every query the application depends on |
| Checkout API response time: 5s | Above threshold | Symptom | Checkout makes multiple DB calls; slow DB → slow checkout |
| Payment API error rate: 8% | Above threshold | Symptom | Slow checkout/DB causes payment calls to time out and fail |
| Orders per minute: 3 | Far below normal | Symptom (effect) | Customers cannot complete slow/failing checkouts, so order volume collapses |

**Analysis — connection diagram:**

```
Database CPU 95% (ROOT CAUSE)
        │
        ▼
Checkout API slow (5s)  ──►  Payment API timeouts (8% errors)
        │                            │
        └──────────────┬─────────────┘
                       ▼
        Orders per minute drops to 3 (business impact)
```

The high database CPU is the **root cause**; the other three are **symptoms** cascading
downstream. Low order volume is an *effect*, not a cause — customers want to buy but the
slow, failing checkout prevents completion.

**Metrics to check next to confirm the hypothesis:**

- Slow-query log / top queries by execution time — is one query or a missing index spiking CPU?
- Database active connections & lock contention — are queries blocking each other?
- Disk I/O wait and memory — is the CPU spike caused by swapping or a full table scan?
- Recent deploys or a marketing/launch event at ~2:00 PM that increased traffic.
- **Three pillars:** traces would pinpoint where checkout spends its 5s; logs would reveal the specific failing queries and payment-timeout errors.

**Proposed remediation plan:**

1. **Immediate:** Identify and kill or throttle the offending query/queries; enable read
   replicas for read-heavy traffic to offload the primary DB.
2. **Short term:** Scale up database resources (vertical) and add the missing index;
   raise payment-API timeout/retry with backoff to reduce cascading failures.
3. **Preventive:** Add query-performance regression tests, capacity-plan for weekend peaks,
   set up autoscaling, and add an alert on slow-query count before CPU saturates.
