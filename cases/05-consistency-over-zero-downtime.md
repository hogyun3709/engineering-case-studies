# 05. Consistency over zero downtime

**System:** cinema booking platform (members, bookings, coupons) moving to a new AWS account
**Stack:** AWS (Elastic Beanstalk, RDS), Python, MySQL

## Problem

The platform had to leave an end-of-life runtime and move to a new AWS account. It held 48 tables, including about 113,000 reservations.

## Decision

Take a short, planned outage and get a clean copy, rather than try for zero downtime without the tooling for it.

- Scaled the old environment to zero to freeze writes, then took the dump.
- Accepted about an hour of errors on the old domain.
- Kept the old database for 30 days as a fallback.

## Verification

- Compared row counts, the latest transactions and API response hashes between the old and new systems.

## After the move

The smaller database then needed protecting at peak times. I chose not to go back to the old managed cluster, add a cache server or resize straight away. Instead I added 30 s caching for public reads, smaller admin pages and an 8 s query limit, with an alarm-based threshold for when to upsize. The old cluster was deleted only after it showed zero connections and a final snapshot existed.

## What I would do differently

The retrospective names a managed migration service and earlier preparation as the way to shorten the outage next time.
