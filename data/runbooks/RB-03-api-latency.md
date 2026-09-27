# API Latency Troubleshooting Runbook

## Overview

This runbook helps you find out why an API has suddenly become slow. It focuses on checkout-api, which serves the website and mobile apps, but the same approach works for any Acme Pay HTTP service. Our target for checkout-api is a p99 latency below 800 ms.

## Step 1: Measure latency correctly

Always start with the **p95 and p99** latency charts, not the average. The average hides the slow requests that customers actually complain about. A p99 of 4 seconds means one request in a hundred takes at least 4 seconds, even if the average still looks fine.

Answer three questions before changing anything:

1. **When did it start?** Note the exact time the latency began to rise.
2. **Is it every endpoint or just one?** Break the latency chart down by endpoint.
3. **Did traffic change?** Compare the request rate with the same time last week.

## Step 2: Every endpoint slow, or just one?

This single question narrows the search more than anything else.

**If every endpoint is slow at the same time**, the cause is almost always something they share:

- the database or its connection pool,
- CPU or memory saturation on the service instances,
- a slow downstream service that every request depends on,
- a large increase in traffic.

**If only one endpoint is slow**, the cause is usually in that endpoint's own code: a new slow query, a missing index, a larger response, or a new call to another service.

## Step 3: Use traces to find the slow part

Distributed traces show how long each part of a request took. Open a few slow traces from the time of the problem and look for the span that grew. Typical findings are:

- a database query that used to take 5 ms and now takes 2 seconds,
- a long wait before the first database query, which usually means the request was waiting for a free connection from the pool,
- a call to payment-service that is waiting for the external card processor.

## Step 4: Check recent changes

Most sudden slowdowns follow a change. Compare the start time of the problem with:

- recent deployments, using `acmectl history <service>`,
- feature flags that were switched on recently,
- configuration changes, such as a smaller connection pool or a lower timeout,
- scheduled jobs, such as reports or data exports that run at a fixed time.

If a deployment lines up with the start of the problem, rolling it back is usually the fastest mitigation. See the Deployment & Rollback Runbook.

## Common causes and what to do

### Database connection pool exhaustion

If traces show requests waiting before their first query, and logs contain `ERR_DB_POOL_TIMEOUT`, the connection pool is exhausted. This was the cause of incident INC-2307. Follow the Database Connection Pool Runbook.

### Slow or new queries

A new query without a suitable index can turn a fast endpoint into a slow one. Check the slow query log on payments-db and run `EXPLAIN ANALYZE` on the worst statement.

### Slow downstream services

payment-service waits up to 10 seconds for the external card processor. If the processor is slow, checkout requests that create payments become slow too. Check the processor status page and the timeout rate on the payment-service dashboard.

### CPU saturation

If CPU usage on the instances is close to 100%, requests queue up. Look for a recent change that made requests more expensive, or add instances with `acmectl instances set <service> --count <n>` as a short-term mitigation.

### Traffic spikes

A marketing campaign or a misbehaving client can multiply traffic. The edge-gateway rate limit protects us from single clients, but a genuine increase in customers may need more instances.

## Related errors: 502, 504 and 429

- **504 Gateway Timeout** from the edge-gateway means the upstream service did not answer within 30 seconds. Treat it as an extreme latency problem and follow this runbook.
- **502 Bad Gateway** right after a release usually means instances were stopped before they finished their open requests.
- **429 Too Many Requests** means a client exceeded its rate limit. This protects the API; it is not a latency problem in itself.

## Mitigation first, root cause second

While customers are affected, focus on making the service fast again: roll back a bad release, switch off a new feature flag, stop an expensive background job, or add instances. Investigate the root cause after the service has recovered, and record what you found in the incident record.
