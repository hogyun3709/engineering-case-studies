# 04. Switch the ledger per channel, not by date

**System:** order and QR ticket ledger shared by a website, merchant shops and kiosks at a theme park in Korea
**Stack:** NestJS, Prisma, MariaDB, Flutter (Windows kiosks)

## Problem

Several sales channels wrote new orders to an older order ledger, and a unified ledger was ready to take over new sales. There was a target date, but a date alone was not allowed to switch anything.

## Decision

Each channel could be switched on and off by itself.

- Each sales channel got its own switch, and new-ledger sales were limited to allow-listed products, with separate package lists for the website and the kiosks.
- The old ledger stayed in service, and scanners and POS read both ledgers, so orders from either side kept working during the move.
- Go-live had automatic no-go checks, and the server refused to move the kiosks to the new ledger unless its readiness gates passed.
- The server treats a missing kiosk setting as "stay on the old ledger". If the setting is unreadable or invalid, new kiosk sales stop instead of guessing.

## What happened

The website, the kiosks and head-office free tickets were switched on the same morning. That day, gaps showed up in how package discount benefits mapped to store products, so the website and kiosk channels were reverted the same afternoon while head-office free tickets stayed on the new ledger.

The rollback only changed switches. No order was moved between ledgers, and orders already sold on the new ledger kept their payments and QR codes. Two web package orders still needed a small manual exception at the front desk.

## Trade-off

Per-channel switches and dual reads are more work to build and to clean up later. They turned a failed go-live into a same-day, switch-only reversal instead of a data migration back.
