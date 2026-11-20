# 09. Limit admission instead of adding capacity

**System:** inventory system for a seafood distributor (office, floor and monitor web apps)
**Stack:** React, FastAPI, SQL Server

## Problem

Under bursts of requests, all 40 workers ended up blocked waiting for database connections, and requests failed or stalled.

## Rejected

- An unlimited connection pool. It moves the queue into the database instead of removing it.
- Resending unclear writes under new IDs. A write whose outcome is unknown can then be applied twice.

## Decision

Admit requests before they reach synchronous dependencies, so a burst no longer leaves every worker blocked on the database.

## Verification

In a controlled burst of 64 requests:

| | Before | After |
|---|---|---|
| Successful requests | 28 | 64 |
| Batch time | 15.7 s | 1.2 s |
