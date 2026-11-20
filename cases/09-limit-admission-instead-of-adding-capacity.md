# 09. Limit admission instead of adding capacity

**System:** inventory system for a seafood distributor (office, floor and monitor web apps)
**Stack:** React, FastAPI, SQL Server

## Problem

In a reproduction with the production pool settings, a burst of read requests took all 40 worker threads. Most of them waited for a database connection, while the requests already holding the connections could not get a thread to finish and give them back. Requests stalled until the pool timeout, and more than half failed.

## Rejected

- An unlimited connection pool or worker count.
- Resending unclear writes under new request IDs. Writes are deduplicated by request ID, so a write whose outcome is unknown could then be applied twice.

## Decision

Admit requests before they reach synchronous dependencies. A small async middleware makes each database request take one of a fixed number of permits (pool size plus overflow, kept below the worker count) before authentication or the endpoint runs. A request that waits past the pool timeout gets a 503 without having held a worker or a connection.

## Verification

In a controlled burst of 64 read requests against an isolated copy with real SQL and the production pool settings:

| | Before | After |
|---|---|---|
| Successful requests | 28 | 64 |
| Batch time | 15.7 s | 1.2 s |

This was a single diagnostic burst, not a sustained load test.
