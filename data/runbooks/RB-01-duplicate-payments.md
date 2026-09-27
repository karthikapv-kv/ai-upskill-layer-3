# Duplicate Payments & Transaction Reversal Runbook

## Purpose

This runbook explains how to investigate a duplicate payment, stop further duplicates, and reverse the extra transaction safely. Use it when a customer reports being charged more than once for the same order, when support escalates a "double charge" ticket, or when the duplicate-ledger-entry alert fires.

Owner: Payments team. Services involved: checkout-api, payment-service, ledger-service and refund-worker.

## How duplicate payments happen

Almost every duplicate payment we have seen falls into one of three patterns:

1. **Retries without a stable idempotency key.** checkout-api retries a request after a timeout but sends a new Idempotency-Key, so payment-service treats the retry as a new transaction. This was the root cause of incident INC-2291.
2. **Double submission from the client.** The customer taps "Pay" twice and the mobile app sends two separate checkout attempts.
3. **Replayed messages.** A payment-events message is processed twice by the ledger-consumers group after a rebalance, creating two ledger entries for one authorisation.

Knowing which pattern you are dealing with tells you whether the problem is isolated to one customer or is affecting many customers right now.

## Step 1: Confirm that it is really a duplicate

Not every pair of similar transactions is a duplicate. Customers sometimes place two genuine orders for the same amount.

- Look up the customer's transactions in the ledger_entries table using their customer ID.
- A true duplicate has the **same order_id**, the **same amount** and the **same card fingerprint**, with timestamps less than 10 minutes apart.
- Check whether both transactions reached the card processor. A transaction in PENDING_UNKNOWN may not have been captured at all; see the payment gateway timeout guide before treating it as a duplicate.
- If the order IDs are different, contact the customer through support before doing anything. It may be two real orders.

## Step 2: Check whether the problem is still happening

Before fixing one customer, make sure you are not in the middle of a wider incident.

- Open the payments dashboard and check the "duplicate ledger entries per minute" panel. A normal day shows zero or one.
- Search the payment-service logs for repeated order IDs in the last hour.
- If more than a handful of customers are affected, or the count is rising, declare an incident using the Production Incident Response Handbook and page the payments on-call engineer.

## Step 3: Stop further duplicates

If duplicates are still being created, contain the problem first:

- If checkout-api retries are the cause, switch off retries with `acmectl flags set checkout-retries --percent 0`. This takes effect within 30 seconds.
- If a recent release introduced the problem, roll it back using the Deployment & Rollback Runbook.
- If ledger-consumers are processing messages twice, pause the consumer group and check for a rebalancing loop in the Kafka Consumer Operations Runbook.

Do not start reversing transactions while new duplicates are still being created, or you will have to repeat the work.

## Step 4: Reverse the duplicate transaction

Once the cause is contained, reverse the extra transaction:

1. Identify the **later** of the two transactions. Always keep the original and reverse the duplicate.
2. Call `POST /ledger/reversals` on ledger-service with the transaction ID of the duplicate and the reason code `DUPLICATE_DEBIT`.
3. ledger-service creates a reversal entry and publishes a refund request to the payment-events topic. The refund-worker picks it up and sends the refund to the card processor.
4. Never delete or edit ledger rows directly in the database. The ledger is append-only, and reversals are how we correct it.

If the reversal call fails with `ERR_LEDGER_409`, a reversal for that transaction already exists. Do not retry; check the existing reversal instead.

For large numbers of affected customers, the payments team runs the bulk reversal script, which applies the same rules. Do not write your own script.

## Step 5: Refund timeline and customer communication

After the reversal, the refund-worker submits the refund to the card processor, normally within a few minutes.

- **Card payments:** the money usually appears back on the customer's statement within **5 to 7 business days**, depending on the customer's bank.
- **Digital wallets:** usually within **1 to 2 business days**.
- Some banks show the original duplicate as "pending" for a few days and then remove it without a refund being needed. Explain this to the customer if they only see a pending transaction.

Tell the customer that the duplicate has been identified and reversed, give them the expected timeline, and add the reversal ID to the support ticket so they can quote it to their bank. If a refund stays in the SUBMITTED state for more than 48 hours, check the refund-worker logs and the processor response.

## Step 6: Record and follow up

- Add a note to the support ticket with the order ID, the duplicate transaction ID and the reversal ID.
- If this was part of an incident, link the ticket to the incident record.
- If you found a new cause of duplicates, raise a ticket for the payments team so a permanent fix can be planned.

## Related documents

- Kafka Consumer Operations Runbook
- Deployment & Rollback Runbook
- Production Incident Response Handbook
