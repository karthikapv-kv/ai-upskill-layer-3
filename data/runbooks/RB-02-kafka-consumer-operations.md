# Kafka Consumer Operations Runbook

## Overview

Several Acme Pay services communicate through Kafka topics. The most important are:

- **payment-events**: published by payment-service whenever a payment is authorised, captured, reversed or refunded. It has 12 partitions.
- **order-events**: published by checkout-api when an order is created, updated or cancelled.

The main consumer groups are **ledger-consumers**, which record every payment event in the ledger, and **notification-consumers**, which send receipts and emails. If ledger-consumers stop, the ledger falls behind and support cannot see recent payments. If notification-consumers stop, customers stop receiving receipts.

Use this runbook when a consumer has stopped processing messages, when consumer lag is growing, or when the consumer lag alert fires.

## First checks when a consumer stops processing messages

Work through these checks in order. Most problems are found in the first three.

1. **Are the consumer instances running?** Check the service health page. If instances are restarting repeatedly, look at their logs for the crash reason before doing anything else.
2. **Is the lag growing on all partitions or only on some?** Open the lag dashboard for the consumer group.
3. **Are the logs full of rebalancing messages?** Repeated "Revoking partitions" and "Joining group" lines mean the group is stuck in a rebalancing loop.
4. **Is one message failing over and over?** The same offset appearing again and again in error logs points to a poison message.
5. **Is a downstream dependency slow?** If the consumer calls payments-db or another service for each message, a slow dependency slows the whole consumer.

## Understanding consumer lag

Consumer lag is the number of messages that have been written to a partition but not yet processed by the consumer group. Some lag is normal during traffic peaks. Lag that keeps growing means the consumers cannot keep up.

- **Lag on one partition only** usually means one consumer instance is stuck, or one slow or failing message is blocking that partition.
- **Lag on every partition** usually means all consumers are too slow, have stopped, or a shared dependency such as the database is slow.

## Rebalancing loops

Kafka expects each consumer to ask for new messages regularly. If a consumer takes longer than `max.poll.interval.ms` to process a batch (300000 ms, or five minutes, in our configuration), the broker assumes the consumer has died and starts a rebalance. During a rebalance no messages are processed at all. If the same slow batch triggers a rebalance every time, the group never makes progress and appears to have stopped completely.

How to fix a rebalancing loop:

- Lower `max.poll.records` so each batch is smaller and finishes faster.
- Find and remove slow work inside the processing loop, such as calls to external services.
- As a last resort, raise `max.poll.interval.ms`, but only after understanding why processing is slow.
- Check whether an instance is crash-looping, because constant restarts also cause repeated rebalances.

## Poison messages and the dead-letter topic

A poison message is a message that fails every time it is processed, for example because it contains invalid JSON or is missing a required field. Without protection, the consumer retries it forever and every message behind it on that partition waits.

Our consumers retry a failing message three times with increasing delays. After the third failure, they publish the message to a dead-letter topic named after the source topic, for example `payment-events.DLT`, and move on to the next message.

- Inspect dead-letter messages with `acmectl kafka dlt-inspect payment-events.DLT`.
- Fix the root cause, which is usually a bug in the producer or a schema change.
- Replay the fixed messages with `acmectl kafka dlt-replay payment-events.DLT`.

Never skip a payment-events message without recording it. Every payment event must eventually reach the ledger.

## Restarting consumers safely

Restarting a stuck consumer instance is safe because offsets are committed only after a message is processed. After a restart, the consumer continues from the last committed offset. This means a message may be processed twice after a restart. The ledger-consumers group handles this by checking the event ID before writing, but other consumers may not, so check with the owning team before restarting unfamiliar consumers.

To restart one instance, use `acmectl restart <service> --instance <id>`. Avoid restarting all instances at once, because it causes a full rebalance.

## Adding consumer instances

If all partitions are lagging because the consumers are simply too slow, add consumer instances with `acmectl instances set <service> --count <n>`. Each partition can be processed by only one consumer in a group, so there is no benefit in running more instances than partitions. For payment-events that limit is 12.

## Escalation

Escalate to the platform team if the Kafka brokers themselves appear unhealthy, if lag keeps growing after the checks above, or if messages are missing rather than delayed. If ledger-consumers have been stopped for more than 15 minutes, declare an incident.
