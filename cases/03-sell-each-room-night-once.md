# 03. Sell each room-night once

**System:** online sales of traditional-inn (ryokan) stays inside a theme park
**Stack:** NestJS, Prisma, MariaDB, NicePay

## Problem

The park was launching online sales of ryokan stays. Two risks mattered most: a room sold twice for the same night, and a price computed in the browser being trusted at payment time.

## Decision

Make one record the authority for occupancy.

- The server issues a signed quote. At checkout it re-checks that quote under a lock, so a client-side total is never trusted.
- Each room-night is claimed once. A second claim for the same room and night fails instead of overbooking.

## Rejected

Dual-reading the old ledger alongside the new one during launch. It would have kept two sources of truth alive. Instead I ran a zero-count audit before launch.

## Launch gate

Launch waited until all 25 readiness blockers on the checklist were cleared.

## Trade-off

The launch came later than a "go live and watch" approach would have. In exchange, there was one occupancy record to trust from day one.
