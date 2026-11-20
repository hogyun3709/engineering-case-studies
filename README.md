# Engineering case studies

Short write-ups of production decisions from systems I built and operate. The code lives in private client and company repositories, so these notes cover what a code sample can't show on its own: the problem, the constraint, the decision, what I rejected, and how I verified the result.

Client names, credentials and security-sensitive details are left out. Numbers come from the project's own measurements, logs and incident notes.

## Payments and money

| # | Case | Decision in one line |
|---|---|---|
| 01 | [Reconcile to the gateway's raw data](cases/01-reconcile-to-raw-gateway-data.md) | Classify payments from raw approval fields, not our own labels, so each revenue total can be checked against an outside record. |
| 02 | [Approval and POS sale are separate facts](cases/02-approval-and-pos-sale-are-separate-facts.md) | A lost POS response is resolved by lookup, never by charging or cancelling again. Devices never fix money on their own. |
| 03 | [Sell each room-night once](cases/03-sell-each-room-night-once.md) | Re-check a signed quote under a lock and claim each room-night once; never trust a client-side total. |

## Rollouts and migrations

| # | Case | Decision in one line |
|---|---|---|
| 04 | [Switch the ledger per channel, not by date](cases/04-switch-the-ledger-per-channel.md) | Per-channel switches, both ledgers readable, automatic no-go checks and an additive rollback. |
| 05 | [Consistency over zero downtime](cases/05-consistency-over-zero-downtime.md) | Freeze writes for a clean move, accept a short outage, keep the old database as a fallback. |

## Data correctness and performance

| # | Case | Decision in one line |
|---|---|---|
| 06 | [Put the duplicate rule in the database](cases/06-put-the-duplicate-rule-in-the-database.md) | Scope the unique key and write permissions to the tenant instead of trusting each sync path. |
| 07 | [Drop a faster query that was wrong](cases/07-drop-a-faster-query-that-was-wrong.md) | Ship 544 ms that matches what users see over 235 ms that sorts on a mostly empty column. |
| 08 | [Shrink backups from measurement](cases/08-shrink-backups-from-measurement.md) | Measure what fills the backup, change the backup plan, and verify before removing anything. |

## Reliability in the field

| # | Case | Decision in one line |
|---|---|---|
| 09 | [Limit admission instead of adding capacity](cases/09-limit-admission-instead-of-adding-capacity.md) | Admit requests before synchronous dependencies rather than growing the connection pool. |
| 10 | [Make the failure visible before fixing it](cases/10-make-the-failure-visible.md) | Get the logs off an unattended kiosk first; one boot failure turned out to have several causes. |

---

More about me: [github.com/hogyun3709](https://github.com/hogyun3709)
