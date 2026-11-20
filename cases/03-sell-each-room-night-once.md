# 03. Sell each room-night once

**System:** online sales of traditional-inn (ryokan) stays inside a theme park in Korea
**Stack:** NestJS, Prisma, MariaDB, NicePay

## Problem

The park was moving ryokan stays to direct online sales. Two risks mattered most: a room sold twice for the same night, and a price computed in the browser being trusted at payment time.

## Decision

Make one record the authority for occupancy.

- The server issues a signed quote. At checkout it locks and re-checks that quote, including that the amount matches, so a client-side total is never trusted.
- Each room-night is claimed once in a single claim table with a unique key per room and night. Online checkouts and staff-entered bookings (phone, walk-in, other platforms) use the same table, so a second claim for the same room and night fails instead of overbooking.

## Rejected

Migrating or dual-reading the old order ledger. Operators confirmed it held no paid ryokan sales, so there was nothing for a dual-read to serve. Instead, a zero-count audit became part of the launch gate: no open room orders in the old ledger, no quotes tied to old-ledger payments, no rows in an older claim table and no pending old-ledger QR jobs. Closed test and refund records stayed read-only.

## Launch gate

Sales were switched on only after the admin readiness check reported zero on all 25 blocker counts (the zero-count audit plus payment, refund, notification and occupancy-consistency checks), with the backup, migration and approved message templates confirmed.

## Trade-off

The launch depended on the readiness check reaching zero, not on a date. In exchange, there was one occupancy record to trust from day one.
