# 07. Drop a faster query that was wrong

**System:** the same multi-tenant contact platform as case 06, campaign dashboard
**Stack:** FastAPI, MySQL

## Problem

A "recent sharers" dashboard query took 41 s on about 80,000 rows.

## First attempt

I got it to 235 ms by sorting on a different column. Then I checked the data: that column was filled for only 315 of the 80,000 rows. The query was fast because the order it produced was mostly meaningless.

## Decision

Revert my own optimization and keep the order users actually saw.

- Shipped a 544 ms query (down from 41 s) that preserves the dashboard's order.
- Moved the per-sharer counts into a cache written at upload time: 15 s+ down to under 1 ms, falling back to the original query on a cache miss.

## Trade-off

544 ms is slower than 235 ms. It is also correct, and the expensive part (the counts) moved to write time, where it belongs.
