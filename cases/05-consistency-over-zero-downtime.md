# 05. Consistency over zero downtime

**System:** cinema booking platform (members, bookings, coupons) moving to a new AWS account
**Stack:** AWS (Elastic Beanstalk, RDS), Python, MySQL

## Problem

The platform had to leave an end-of-life runtime and move to a new AWS account. It held 48 tables, including about 113,000 reservations.

## Decision

Take a short, planned outage and get a clean copy, rather than try for zero downtime without the tooling for it.

- Scaled the old environment to zero to freeze writes, then took the final dump.
- Accepted about an hour and a half of errors on the old domain, from stopping the old backend to switching DNS.
- Kept the old database running as a fallback, with deletion not considered until 30 days after the move.

## Verification

- Checked row counts for the main tables and the latest sign-up, reservation and payment times in the new database against the end of old operation.
- After the DNS switch, confirmed the old and new API domains returned identical responses (hash match, timestamps excluded).

## After the move

Later, to cut cost, the database moved again, from a serverless cluster to a smaller fixed-size instance. That instance then needed protecting against expected peaks such as holidays and big releases. I chose not to go back to the serverless cluster, add a cache server or resize straight away. Instead I added 30 s in-process caching for public reads, capped admin page sizes and set an 8 s limit on read queries, with alarm-based criteria for when to move up one size. The serverless cluster was deleted only after it showed zero connections and a final snapshot existed.

## What I would do differently

The retrospective names a managed migration service and earlier preparation as the way to shorten the outage next time.
