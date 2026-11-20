# 04. Switch the ledger per channel, not by date

**System:** order and QR ticket ledger shared by a website, merchant shops and kiosks
**Stack:** NestJS, Prisma, MariaDB, Flutter (Windows kiosks)

## Problem

Several sales channels wrote to an older order ledger, and a unified ledger was ready to replace it. The simple plan was a cut-over date for everything at once.

## Decision

No single cut-over date. Each channel moved on its own terms.

- Each sales channel got its own switch and its own product allow-list.
- Both ledgers stayed readable side by side, so either could be checked during the move.
- Go-live had automatic no-go checks.
- Installed kiosks treat a missing server setting as "stay on the old flow". The switch to the new flow waited for a physical end-to-end test at real stores.

## What happened

The first go-live was reverted the same day. Because the rollback was additive, the old flow came back without moving or losing a single order, and the kiosks returned to the old flow as well.

## Trade-off

Per-channel switches and dual reads are more work to build and to clean up later. They turned a failed go-live into a same-day reversal instead of a data repair.
