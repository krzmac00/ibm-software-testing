# Create a Performance Plan for a Backend System

## Scenario

I work at **QuickShop**, an online shopping website. Customers are complaining that the
website is slow. My manager asked me to create a simple performance plan to fix the issues.

**Current problems:**

- Product search takes 6 seconds
- Checkout page is slow
- Website crashes when many users visit at the same time

---

## Exercise 1: The Three Performance Layers

Every backend system has three layers where performance problems can occur:

1. **Database layer** — Where data is stored and retrieved
2. **Application layer** — Where the code runs
3. **Infrastructure layer** — The servers and hardware

---

## Exercise 2: Create the Performance Plan

### Task A: Fix database layer problems

**Problem:** Product search takes 6 seconds because the database scans all 50,000 products.

| What's the problem? | Why is it slow? | What should we do? | How much faster? |
|---|---|---|---|
| Product search | No indexes on product_name column | Add index on product_name | 80–90% faster (under 1 second) |
| Slow product listing | Queries use `SELECT *` and pull every column | Select only needed columns (name, price, image) | 30–50% less data transferred, faster queries |
| Same products fetched repeatedly | No caching — DB hit on every request | Cache popular search results / product data in Redis | Near-instant for cached items, big DB load drop |

### Task B: Fix application layer problems

**Problem:** After users click "Buy Now," they wait 12 seconds before seeing the
confirmation page because the system sends a confirmation email during checkout.

| What's the problem? | Current wait time | What should we do? | How much faster? |
|---|---|---|---|
| Email slows down checkout | 12 seconds | Send email in background (asynchronously) via a queue | Instant confirmation page |
| Many separate API calls | ~4 seconds | Batch the calls into one request | 60–70% faster |
| New DB connection per request | ~1 second overhead | Reuse connections with a connection pool | Removes per-request connection delay |

### Task C: Fix infrastructure layer problems

**Problem:** The website crashes when 5,000 users visit at the same time because there is
only one server.

| Current setup | What's the problem? | What should we do? | What type of scaling? |
|---|---|---|---|
| 1 server | Crashes during high traffic | Add 2 more servers + load balancer | Horizontal scaling |
| Single server, limited RAM/CPU | Runs out of resources at busy times | Upgrade the server to more RAM/CPU | Vertical scaling |
| No traffic distribution | All users hit one machine | Add a load balancer to spread requests | Horizontal scaling |

---

## Exercise 3: Prioritize Using the 80/20 Rule

The 80/20 rule says **20% of the optimizations give 80% of the improvement** — fix the
biggest problems first.

### Step 2: Top 3 priorities

| Priority | What to fix | Which layer? | Why fix this first? |
|---|---|---|---|
| 1 | Add database indexes for search | Database | Most users search for products — this affects everyone, and it's a quick high-impact win |
| 2 | Add servers + load balancer | Infrastructure | Crashes during traffic spikes mean total downtime and lost sales for *all* users at once |
| 3 | Send confirmation email asynchronously | Application | Cuts checkout from 12s to instant, directly improving conversion at the revenue moment |

### Step 3: Understand trade-offs

| Solution | Good things | Bad things | Is it worth it? |
|---|---|---|---|
| Add 2 more servers | Can handle more users, prevents crashes | Costs more money, more complex | Yes — lost sales cost more than servers |
| Use caching | Much faster, less database load | Need to keep cache updated (staleness risk) | Yes — simple to implement |
| Add database index | Search drops from 6s to <1s, helps everyone | Slightly slower writes, uses extra disk space | Yes — read speedup far outweighs write cost |
| Async email sending | Instant checkout, better conversion | Need a queue/worker; harder to debug failures | Yes — checkout speed directly drives revenue |

> **Key insight:** The weakest link determines your system's performance, not the strongest link.

---

## Challenge 1: New Problem to Solve

**Scenario:** QuickShop wants to add product reviews. Each product page shows 50 reviews,
meaning 50 extra database queries every time someone views a product.

1. **Which layer will have problems?**
   The **Database layer** — 50 extra queries per page view multiplies database load
   enormously on a popular product.

2. **What optimization technique would you use?**
   **Caching** (e.g., store the reviews for each product in **Redis**). Also fetch all 50
   reviews in a **single batched query** instead of 50 separate ones.

3. **Why is that the best solution?**
   Reviews change rarely — most page views see the exact same reviews — so they are ideal
   for caching. Serving them from Redis means the database is queried once (when the cache
   is populated or a new review is added) instead of 50 times per view. This removes almost
   all the load while keeping the page fast, and the cache only needs refreshing when a new
   review is posted.

---

## Challenge 2: Performance Investigation

**Scenario:** All optimizations were implemented and the site ran great for 2 months.
Suddenly product search is slow again (5 seconds), but nothing changed in the code.

**Two possible reasons:**

1. **The data has grown.** Many more products were added over two months. The index may now
   be much larger, or query patterns that were fine at 50,000 products no longer scale at,
   say, 500,000 — the index might need rebuilding, re-tuning, or partitioning.

2. **The cache stopped working.** The Redis cache may be down, full (evicting entries), or
   misconfigured, so every search now falls through to the database. Alternatively, the
   index is no longer being used — it could have become fragmented or the query planner
   stopped choosing it after statistics drifted.

> **Takeaway:** **Monitoring** (query times, cache hit rate, database size, index
> usage) is needed to detect and diagnose this kind of gradual regression before customers do.
