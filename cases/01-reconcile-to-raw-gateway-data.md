# 01. Reconcile to the gateway's raw data

**System:** commerce platform for a theme park in Korea (tickets, stays, memberships, merchant sales)
**Stack:** NestJS, Prisma, MariaDB, NicePay (cards, KakaoPay, Naver Pay, prepaid money, virtual accounts)

## Problem

Card revenue in our admin dashboard did not match the card sales the tax authority receives from the payment gateway, which reports them as our payment agent. For one month the dashboard was 67,500 KRW higher.

## What I found

The gap was one wallet payment (KakaoPay Money) that the gateway returns in a card-shaped response. Our order record stored it as a card payment, so the dashboard counted it as card revenue. Wallet payments are not part of the gateway's card-sales filing; the wallet provider settles them.

## Decision

Classify each gateway payment from the gateway's raw approval fields (card, KakaoPay Money, Naver Pay points, Toss money, virtual account, bank transfer) instead of the method our system stored at checkout.

- Classification happens when orders are read, so no schema migration was needed.
- A follow-up fix made every new online order store those raw approval fields.
- Older orders that lack them fall back to the stored card-company name, which only recognises wallet names.

## Trade-off

Reading raw fields is more code than trusting one enum, and the fallback for older orders is less precise. I accepted both, because a dashboard that cannot be checked against outside records is worse than a slightly more complex one.

## Verification

- Unit tests use raw gateway responses copied from real orders of that month, including the wallet payment that caused the gap.
- The dashboard now shows separate totals for card, bank transfer and each wallet. Each total can be checked against one outside record: the gateway's tax-authority filing or that wallet provider's settlement statement. Order lists show each order's resolved method.
