# 07. Drop a faster query that was wrong

**System:** the same multi-tenant contact platform as case 06, campaign contacts page
**Stack:** FastAPI, MySQL

## Problem

The first version of a "most recent sharer first" sort on a campaign's contact list (about 80,000 contacts) took 41 s.

## First attempt

Rewriting the query and adding a covering index brought it to 2.9 s. Sorting on a stored "last shared" column instead brought it to 235 ms. Then I checked the data: that column was filled for only 315 of the 80,000 rows. The query was fast, but the order it produced was mostly meaningless.

## Decision

Drop my own optimization and match the order of the sharer list users already saw.

- Shipped a 544 ms query (down from 41 s) that sorts by each sharer's latest upload, the same source as the sharer list. A 377 ms version in between was dropped for the same reason: it sorted on a timestamp that a backfill had set to one value for every sharer.
- Moved the per-sharer counts into a cache table written at upload time: 15 s+ down to under 1 ms, falling back to the original query on a cache miss.

## Trade-off

544 ms is slower than 235 ms. It is also correct, and the expensive part (the counts) moved to write time, where it belongs.
