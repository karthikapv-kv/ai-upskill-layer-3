# Deployment & Rollback Runbook

## Overview

This runbook describes how Acme Pay services are deployed to production and how to roll back a failed deployment. All deployments and rollbacks use the `acmectl` command-line tool, which records every action in the deployment history.

## Before you deploy

Check each of these before starting a production deployment:

- The CI pipeline for the release is green, including integration tests.
- An approved change ticket exists for the release.
- You know which version is currently running. Check with `acmectl history <service>`.
- If the release contains a database migration, the migration has been reviewed and is **backwards compatible** with the currently running version.
- There is no release freeze in place, and it is not after 15:00 on a Friday if you are deploying payment-service or ledger-service.
- Someone else on the team knows you are deploying and is available if something goes wrong.

## How to deploy

Start the deployment with:

```
acmectl deploy <service> --version <version>
```

The deployment then goes through two stages automatically.

### Canary stage

The new version is first deployed to a **canary group of 5% of instances** for **15 minutes**. During this time, acmectl compares the canary's error rate and p99 latency with the instances still running the previous version. If the canary is clearly worse, the deployment stops automatically and the canary instances return to the previous version.

### Rollout stage

If the canary is healthy, the rollout continues in batches of **25% of instances**, with a short health check between batches. A typical deployment finishes in about 30 minutes.

## Watching a deployment

Stay with your deployment until it has finished and been healthy for at least 15 minutes. Watch:

- the error rate and p99 latency of the service,
- business metrics such as successful payments per minute,
- new error messages in the logs,
- the decline rate, if you are deploying payment-service. A sudden rise in card declines after a release usually means requests are malformed.

## When to roll back

Roll back **first** and investigate **afterwards** if you see any of the following after a deployment:

- error rate clearly higher than before the release,
- p99 latency clearly higher than before the release,
- a drop in successful payments or orders,
- any data correctness problem, such as duplicate ledger entries.

Do not spend time debugging in production while customers are affected. A rollback is fast and safe in most cases.

## How to roll back a failed deployment

1. Find the previous version with `acmectl history <service>`.
2. Run:

   ```
   acmectl rollback <service> --to <previous-version>
   ```

3. A rollback **skips the canary stage** and redeploys the previous version to all instances at once. It normally completes within about **five minutes**.
4. Confirm that error rate and latency have returned to normal.
5. Post a message in the incident channel saying which service was rolled back, from which version to which version, and why.

## Rollbacks and database migrations

acmectl does **not** roll back database migrations. This is the most common reason a rollback goes wrong.

- If the migration only **added** tables or columns, the old version will normally keep working, and you can roll back safely.
- If the migration **removed or renamed** columns that the old version still uses, rolling back the code will break the old version. In this case, do not roll back. Instead, switch off the new behaviour with its feature flag, or deploy a fix forward.

This is why every migration must be backwards compatible with the version before it.

## Feature flags as an alternative to rollback

New behaviour should ship behind a feature flag. Switching a flag off takes effect within 30 seconds and does not require a deployment:

```
acmectl flags set <flag-name> --percent 0
```

If the problem is clearly caused by a flagged feature, switching the flag off is faster and safer than a full rollback.

## After a rollback

- Keep the failed version from being redeployed until the cause is understood.
- Create a ticket describing what went wrong, with links to dashboards and logs.
- If customers were affected, follow the Production Incident Response Handbook and write a postmortem.
