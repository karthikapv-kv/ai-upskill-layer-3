# Database Connection Pool Runbook

## Overview

Acme Pay services store their data in a PostgreSQL cluster called **payments-db**. Opening a new database connection is slow, so each service instance keeps a **connection pool**: a fixed set of open connections that requests borrow and return. When the pool works well, requests get a connection instantly. When every connection is busy, new requests have to wait.

Use this runbook when you see `ERR_DB_POOL_TIMEOUT` errors, when latency rises across all endpoints of a service at once, or when the pool usage alert fires.

## How our pool is configured

Every service uses the same pool settings, which can be changed per service:

- `db.pool.max_size`: the maximum number of connections per instance. The default is **20**.
- `db.pool.acquire_timeout_ms`: how long a request waits for a free connection before failing. The default is **2000 ms**.
- `db.pool.idle_timeout_ms`: how long an unused connection stays open before it is closed. The default is 10 minutes.

When a request has waited `db.pool.acquire_timeout_ms` without getting a connection, it fails with `ERR_DB_POOL_TIMEOUT`.

## Symptoms of pool exhaustion

- Latency rises on **every** endpoint of the service at the same time, including endpoints that are normally fast.
- Traces show a long gap before the first database query in each request.
- Logs contain `ERR_DB_POOL_TIMEOUT`, usually after latency has already been high for a while.
- The pool usage panel on the service dashboard is at or near 100%.
- payments-db itself may look quiet, because the problem is waiting for connections, not database load.

## Diagnosis

### Step 1: Confirm the pool is full

Check the pool usage panel for the affected service. If it is at 100% on every instance, the pool is exhausted. If only one instance is affected, that instance may have a connection leak.

### Step 2: Find out what is holding the connections

Run this query on payments-db to see what the connections are doing:

```sql
SELECT application_name, state, now() - query_start AS duration, query
FROM pg_stat_activity
WHERE datname = 'payments'
ORDER BY duration DESC;
```

Look for:

- **Long-running queries**: a query that holds a connection for seconds instead of milliseconds.
- **"idle in transaction"** connections: code that opened a transaction and never committed or rolled it back.
- A single application using far more connections than usual.

### Step 3: Check recent changes

Pool problems are usually caused by a change: a new query, a new background job, a release that forgot to close connections in an error path, or a configuration change that reduced `db.pool.max_size`.

## Common causes

1. **Slow queries holding connections.** A slow report or a query without an index keeps each connection busy for a long time, so the pool drains quickly. This was the cause of incident INC-2307, where a new reporting query exhausted the pool on every checkout-api instance.
2. **Connection leaks.** Code that borrows a connection and does not return it when an error occurs. Usage creeps up over hours until the pool is full.
3. **Long transactions.** Calling an external service, such as the card processor, while holding a database transaction open.
4. **Too many instances.** payments-db accepts at most **400** connections in total. The number of instances multiplied by `db.pool.max_size` must stay below that limit across all services.

## Fixes

- **Stop the source of slow queries.** Pause the offending job or roll back the release that introduced the query.
- **Restart affected instances** to release leaked connections. This is a short-term fix only; the leak will return.
- **Add a missing index** if a slow query is scanning a large table. Create indexes concurrently in production.
- **Move reporting queries to the read replica** so they cannot starve customer-facing requests.

## Why raising the pool size is rarely the answer

It is tempting to increase `db.pool.max_size` when the pool is full. This usually makes things worse: more connections mean more work for payments-db at the same time, and the total can exceed the 400-connection limit, causing failures in other services. Raise the pool size only when you have confirmed that queries are fast, nothing is leaking, and the service genuinely needs more concurrent database work.

## Prevention

- Alert when pool usage stays above 80% for five minutes.
- Review the slow query log weekly.
- Never call external services inside a database transaction.
- Run reporting and analytics queries against the read replica only.
