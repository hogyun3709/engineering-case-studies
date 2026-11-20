# 06. Put the duplicate rule in the database

**System:** multi-tenant contact management and messaging platform used by campaign teams
**Stack:** FastAPI, MySQL, React, Flutter

## Problem

Contacts shared from a mobile app reached one table through several write paths (single upload, chunked upload, staged upload and a background merge job), all using upserts. When the same person shared the same contact in two campaigns, the second upload overwrote the record that belonged to the first campaign.

## Cause

The unique key that every upsert relied on was (sharer, contact) and did not include the campaign. Whichever path ran, the upsert matched the other campaign's row.

## Decision

Fix the rule where every path meets it: the database.

- Added the campaign ID to the unique key.
- In the same change, made the background merge job's upsert match the upload path.

Earlier and separately, I had added a campaign write-permission check to 30 write endpoints. It blocks read-only admins and admins without access to that campaign, and has unit tests.

## Related: uploads that can resume

The first staging-table upload merged everything in one transaction (5,000 rows in testing) and deadlocked when several merges ran at once. I split the merge into 1,000-row batches. Each batch is its own transaction and deletes the staging rows it merged, so locks are held briefly and a retried finalize continues with the rows that remain. At most five merges run at once, for a target of 20 people uploading together. A merge waits up to 55 s for a slot, inside the 60 s gateway limit, instead of failing at once.

## Trade-off

A schema change on a live multi-tenant table needs more care than patching each sync path. It is also the only fix that covers sync paths added later.
