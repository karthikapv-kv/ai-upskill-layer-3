# Production Incident Response Handbook

## Purpose

This handbook describes how Acme Pay engineers respond to production incidents: how to decide how serious an incident is, who does what, how we communicate, and how we learn from incidents afterwards. Service-specific troubleshooting lives in the individual runbooks; this handbook covers the process around them.

## What counts as an incident

Declare an incident when customers are affected or are likely to be affected soon. Examples include failed or duplicate payments, checkout errors, very slow APIs, missing receipts, or the ledger falling behind. When in doubt, declare an incident. It is easy to close one that turns out to be minor, and expensive to start one too late.

Declare an incident by running `acmectl incident open --title "<short description>"`. This creates an incident record, an incident channel and an incident number such as INC-2291.

## Severity levels

- **SEV1**: most customers cannot pay or check out, or money is being moved incorrectly at scale. Page the on-call engineers immediately and inform engineering leadership.
- **SEV2**: a significant feature is broken or badly degraded for many customers, for example very slow checkout or delayed refunds.
- **SEV3**: a minor problem with a workaround, affecting a small number of customers.

Any incident where customers are charged incorrectly is at least SEV2.

## Roles

- **Incident commander**: coordinates the response, decides on actions, and keeps track of what has been tried. The incident commander does not debug.
- **Operations lead**: investigates and applies fixes, often with help from other engineers.
- **Communications lead**: posts regular updates for support and for internal stakeholders.

In a small incident one person may hold more than one role, but for SEV1 and SEV2 incidents the incident commander should be a separate person.

## The first 15 minutes

1. Declare the incident and assign an incident commander.
2. Write down what is known: symptoms, start time, affected services and an estimate of affected customers.
3. Check what changed recently: deployments, feature flags, configuration changes and scheduled jobs.
4. Choose a mitigation and apply it.
5. Post the first update in the incident channel.

## Mitigate first

The first goal is to stop customer impact, not to find the root cause. Common mitigations are:

- **Roll back** a recent deployment using the Deployment & Rollback Runbook.
- **Switch off** a recently enabled feature flag.
- **Stop** an expensive background job or report.
- **Restart** stuck service or consumer instances.
- **Add instances** to a service that is overloaded.
- **Disable retries** that are making a problem worse, as in INC-2291.

Record every action in the incident channel with the time it was taken, so the timeline can be reconstructed later.

## Communication

- Post an update in the incident channel at least every 30 minutes for SEV1 and every hour for SEV2, even if nothing has changed.
- Tell support what customers are experiencing and what they should say to customers.
- Do not guess at root causes in public updates. Describe impact and progress.

## Closing an incident

Close an incident when customer impact has stopped and the system has been stable for at least 30 minutes. Clean-up work, such as reversing duplicate transactions or replaying dead-letter messages, can continue after the incident is closed, but must be tracked with tickets.

## Postmortems

Every SEV1 and SEV2 incident needs a written postmortem within five business days. A postmortem is blameless: it describes what happened, why the system allowed it to happen, and what we will change. It contains a timeline, the root cause, what went well, what went badly, and follow-up actions with owners.

## Recent incidents

### INC-2291: duplicate charges after a retry storm

A network slowdown between checkout-api and payment-service caused client timeouts. checkout-api retried each timed-out request, but a bug generated a new idempotency key for every retry, so payment-service treated each retry as a new transaction. About 1,800 customers were charged two or three times over 40 minutes. Mitigation: retries were switched off with the checkout-retries feature flag, and the duplicates were reversed using the Duplicate Payments & Transaction Reversal Runbook. Follow-up: reuse idempotency keys across retries, and alert on duplicate ledger entries.

### INC-2307: checkout latency spike from connection pool exhaustion

During a sales campaign, checkout-api p99 latency rose from 300 ms to 9 seconds. A new reporting query held database connections for several seconds each, which exhausted the connection pool on every checkout-api instance. Requests waited for a free connection and then failed with ERR_DB_POOL_TIMEOUT. Mitigation: the reporting job was stopped and checkout-api was restarted. Follow-up: move reporting queries to the read replica, add an index on ledger_entries.order_id, and alert when pool usage stays above 80%.
