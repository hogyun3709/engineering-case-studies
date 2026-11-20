# 08. Shrink backups from measurement

**System:** daily database backups to object storage for the theme park platform
**Stack:** MariaDB, Cloudflare R2

## Problem

Backup storage and backup time kept growing. The quick fixes on the table were deleting old rows or dropping audit logs.

## Measurement first

One log outbox table made up about 95% of backup size and 70% of backup time.

## Decision

Change the backup plan, not the data.

- Weekly full backups plus smaller partial backups in between.
- No rows deleted and no audit logs dropped.

## Verification before cleanup

Before removing any old backup sets, I checked the newest one row by row: 1.43 M rows, with checksums.

## Result

Backup storage went from 4.31 GB to 169 MB.
