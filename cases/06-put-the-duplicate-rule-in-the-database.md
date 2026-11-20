# 06. Put the duplicate rule in the database

**System:** multi-tenant contact management and messaging platform used by campaign teams
**Stack:** FastAPI, MySQL, React, Flutter

## Problem

Contacts arrived through several collection channels and were synchronized with upserts. Records from one team were overwriting records from another team.

## Cause

The unique key that every upsert relied on did not include the team (campaign) scope. Each sync path was correct on its own; the shared rule underneath was not.

## Decision

Fix the rule where every path meets it: the database.

- Added the campaign ID to the unique key.
- Enforced campaign-scoped write permission on 30 endpoints, with permission tests.

## Related: uploads that can resume

Large contact uploads ran as one 5,000-row transaction. I split them into 1,000-row batches through a staging table, so 20 concurrent uploaders hold locks only briefly and a retried finalize continues where it stopped. Uploads wait up to 55 s for a slot, inside the 60 s gateway limit, instead of failing at once.

## Trade-off

A schema change on a live multi-tenant table needs more care than patching each sync path. It is also the only fix that covers sync paths added later.
