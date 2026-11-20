# 08. Shrink backups from measurement

**System:** nightly database backups to object storage for a theme park platform in Korea
**Stack:** NestJS, MariaDB, Cloudflare R2

## Problem

Backups had no retention, so old backup sets kept piling up in object storage, and every nightly run re-read one large table in full. The quick fixes on the table were deleting old rows or dropping audit logs.

## Measurement first

In one nightly backup, a single log outbox table made up about 95% of the compressed backup size and about 70% of the backup time.

## Decision

Change the backup plan, not the business records.

- One-time cleanup: keep only the newest backup set and remove the older ones.
- A full backup once a week. On other nights, only that outbox table is backed up partially (recent, undelivered or still-carrying-payload rows); every other table is still backed up in full. If no verified full backup exists from the last six days, the run becomes a full backup.
- Automatic retention after each nightly run, which verifies the sets it keeps before deleting anything and never deletes the newest full backup.
- No order, payment or audit records were deleted to make backups smaller.

## Verification before cleanup

Before removing any old backup sets, I checked the newest one end to end: every table file decompressed, every row parsed, 1.43 M rows matched, with checksums.

## Result

The one-time cleanup took backup storage from 4.31 GB to 169 MB. The weekly-full schedule and automatic retention came afterwards, so old sets would not pile up again.
