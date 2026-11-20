# 01. Reconcile to the gateway's raw data

**System:** commerce platform for a theme park in Korea (tickets, stays, memberships, merchant sales)
**Stack:** NestJS, Prisma, MariaDB, NicePay (cards, KakaoPay, Naver Pay, prepaid money, virtual accounts)

## Problem

Revenue in our admin dashboard did not match the sales records the payment gateway files with the tax authority as our payment agent. Both sides looked internally consistent, so the question was which label was wrong.

## What I found

The gap came from a wallet payment (KakaoPay Money) that the gateway returns in a card-shaped response. Our own order records labelled it as a card, so the dashboard grouped it with card sales while the outside records did not.

## Decision

Classify every payment from the gateway's raw approval fields (card, KakaoPay Money, Naver Pay points, Toss money, virtual account) instead of the label our system assigned at checkout.

- Classification happens when orders are read, so no schema migration was needed.
- Every new order keeps the raw approval fields.
- Older orders that lack them fall back to the card-company name.

## Trade-off

Reading raw fields is more code than trusting one enum, and the fallback for older orders is less precise. I accepted both, because a dashboard that cannot be checked against outside records is worse than a slightly more complex one.

## Verification

- Tested against real orders from one month, not only fixtures.
- The dashboard can now be compared line by line with the gateway's records and the wallet providers' statements.
