# System Design - Payment System

FIRST CREATED 2022/07/06

UPDATED 2022/07/19

## Functional Requirements

**Need to clarify -** Do we use 3rd party payment processors or we have to design one? In this chapter, we use 3rd external such as Stripe. We are building a payment backend for an e-commerce application and we are going to use 3rd party payment processors.

Pay-in-flow : payment service receives money from customers on behalf of sellers.

Pay-out-flow : payment service sends money to sellers.

Take the e-commerce site as an example, [**Buyer**] ->(Pay-in)-> [**E-commerce Website**] ->(Pay-out)-> [**Seller**]

## Non-Functional Requirements

Reliability and fault tolerance.

Need to perform reconciliation and fix any inconsistencies.

## Calculation

1 million transactions per day.

## Components

Payment Service - accepts payment event from users and coordinates the payment process.

Payment Executor - executes a single payment order via a PSP.

Payment Service Provider (PSP) - 3rd party payment provider like Paypal, Stripe, etc.

## APIs

### POST /v1/payments

Keep the data type of the "amount" field in string format instead of integer or double.

A single payment event can contain multiple payment orders/

### GET /v1/payments/{:id}

Retrieve the execution status of a single payment order based on payment_order_id.

## Table Schema

### Payment Event Table

checkout_id (PK), buyer_info, seller_info, credit_card_info, is_payment_done

### Payment Order Table

payment_order_id (PK), buyer_account, amount (string), currency, checkout_id (Foreign Key), payment_order_status, ledger_updated, wallet_updated

payment_order_status: enum type of (NOT_STARTED, EXECUTING, SUCCESS, FAILED)

## Double-Entry Ledger System

TODO

## Reference

[1] System Design Interview Book Volume II. Alex Xu. Chapter 11. Payment System.
