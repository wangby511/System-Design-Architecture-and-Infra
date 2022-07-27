# System Design - Payment System

FIRST CREATED 2022/07/06

UPDATED 2022/07/26

## Functional Requirements

**Need to clarify -** Do we use 3rd party payment processors or we have to design one? In this chapter, we use 3rd external such as Stripe. We are building a payment backend for an e-commerce application and we are going to use 3rd party payment processors.

**Pay-in-flow** - payment service receives money from customers on behalf of sellers.

**Pay-out-flow** - payment service sends money to sellers.

Take the e-commerce site as an example, [**Buyer**] ->(Pay-in)-> [**E-commerce Website**] ->(Pay-out)-> [**Seller**]

## Non-Functional Requirements

Reliability and fault tolerance.

Need to perform reconciliation and fix any inconsistencies.

## Calculation

1 million transactions per day = 10 transactions per second (TPS).

## Components

Payment Service - accepts payment event from users and coordinates the payment process. Orchestrator.

Payment Executor - executes a single payment order via a PSP.

Payment Service Provider (PSP) - 3rd party payment provider like Paypal, Stripe, etc. Usually PSP provides a hosted payment page to collect card payment details.

## APIs

### POST /v1/payments

Keep the data type of the "amount" field in string format instead of integer or double.

Executes a payment event. A single payment event can contain multiple payment orders.

### GET /v1/payments/{:id}

Retrieve the execution status of a single payment order based on **payment_order_id**.

## Table Schema

We take traditional database with ACID transaction.

### Payment Event Table

checkout_id (PK), buyer_info, seller_info, credit_card_info, is_payment_done

### Payment Order Table

payment_order_id (PK), buyer_account, amount (string), currency, checkout_id (Foreign Key), payment_order_status, ledger_updated, wallet_updated

payment_order_status: enum type of (NOT_STARTED, EXECUTING, SUCCESS, FAILED)

When the **payment_order_status** is SUCCESS, the payment service calls the wallet service to update the seller balance and update the **wallet_updated** field to TRUE.

Once it is done, the next step is to call the ledger service to update the ledger database by updating the **ledger_update** field to TRUE.

When all the payment orders under the same checkout_id are processed successfully, the payment service updates the **is_payment_done** to TRUE in the payment event table.

## Double-Entry Ledger System

TODO

## Reconciliation

This is a practice that periodically compares the states among related services in order to verify that they are in agreement, and also to detect some discrepancy/mismatch difference.

E.g. every night PSP or banks send a **settlement** file to their clients.

## Payment Processing Delays

There are two ways of handling the async result of payment (if the request takes a long time to process):

1. The PSP return a pending status to our client.

2. The PSP tracks the pending payment on our behalf and notifies our backend payment service via the **webhook**. The **webhook** is an URL on the payment system side that was registered with the PSP during the initial setup with the PSP.

## Asynchronous Communication

Usually we use async communication for a large-scale payment system to trade design simplicity and consistency for scalability and failure resilience. It is able to **de-couple and scale**. Also we can have **failure isolation**.

**We usually have a message queue**. There are 2 modes: Single receiver and Multiple receivers.

1) Single receiver - Each message/request is processed by only one receiver or service.

2) Multiple receivers - Each message/request is processed by multiple consumers like Payment system, Analytics and Billing.

## Error Handling

For retry-able failures, they are routed to **retry queue**. Otherwise for non-retry-able failures, such as invalid inputs, stored in the database.

If the retry count reaches the maximum threshold, the event is put into **dead letter queue**.

Retry strategy - Immediate retry, Fixed intervals, Incremental intervals, Exponential back-off, Cancel.

## Idempotency

Idempotency is the key to ensuring the at-most-once guarantee.

From an API standpoint, idempotency means clients can make the same call repeatedly and produce the same result. To perform an idempotent payment request, an idempotency key is added to the HTTP header as <idempotency-key : key_value>.

What if the user clicks the "pay" button quickly twice? - For the second request, it is considered as a retry of the first one. Since the server has already seen the idempotency key, it does no operation and just return previous message. The idempotency key could be shopping cart id.

What if the PSP processes the payment successfully but the response fails to reach our backend system? - Since the payment order remains the same, the token used as idempotency key is still the same. The PSP is able to identify the double payment and return the result of previous execution.

### Database of Storing Idempotency Keys

In a system of distributed databases, there is a tradeoff between consistency and latency.

Since we couldn’t tolerate high latency or reading uncommitted data, **only using the primary(master) database** made the most sense for us. In doing so, there is **no need for using a cache or a database replica**. Because if the read is hit on replica due to a replication lag from primary database to secondary replicas, the service could mistakenly execute the payment again, resulting in duplicate payments.

When **using a single master database for idempotency**, it was pretty clear that scaling would undoubtedly and quickly becomes an issue. To avoid the scaling problem of this single primary(master) database approach, we can do sharding the database by idempotency key.

## Reference

[1] System Design Interview Book Volume II. Alex Xu. Chapter 11. Payment System.

[2] [Avoiding Double Payments in a Distributed Payments System - The Airbnb Tech Blog](https://medium.com/airbnb-engineering/avoiding-double-payments-in-a-distributed-payments-system-2981f6b070bb)
