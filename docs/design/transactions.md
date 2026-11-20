# Transaction protocol and race rules

Publish locks one routing snapshot and all destination admission decisions in one canonical transaction. Claim protects shared capacity and authority together. ACK/NACK/expiry/redrive each commits its event and receipt with the state transition. Consumer effects use a separate store and deduplicate by queue-entry identity.

## Shared accepting protocol

1. Bound/parse input and prepare canonical bytes outside the writer.
2. Acquire the write transaction before reading mutable authority, admission or shared limits.
3. Find the scoped receipt. Same digest returns its original committed outcome; changed content returns conflict.
4. Reload current generation, expected revision/epoch, phase and database time. Check every related precondition in this snapshot.
5. Apply the state transition with conditional source state; enforce unique logical identities and finite limits.
6. Allocate ordered audit/feed events and write response receipt in that same transaction. Commit before confirmation.

The decisive conditional mutation is the linearization point if the transaction commits. Another writer cannot interleave. Deadlines use database-evaluated wall-clock UTC; equality means expired. Read time after lock acquisition and again immediately before authority-dependent writes. Commit response time is not the authority check time. Clock jumps remain a documented machine-clock limitation.

## Lock order and retries

Use receipt, parent aggregate, sorted child IDs, authority/position metadata, outcome rows, audit, then receipt write. SQLite serializes writers; this order keeps the proof reviewable and avoids inverted access if the store later changes. No network call or reducer/handler runs under a write lock.

Retry whole transactions on documented transient busy/conflict results, with bounded backoff outside the lock (proposed 10/25/50/100/200 ms and a two-second request budget). Roll back fully; never retry only a suffix of SQL statements. Exhaustion returns 503. A lost COMMIT response may still have committed: discard the uncertain connection and replay the same key through a fresh one.

## Read consistency and capacity

Read bounds, records, revisions and event high-water mark in one short read snapshot. Keyset pagination has a stable ID tie-breaker and filter-bound cursor. Capacity counts come from authoritative rows inside the same writer as admission; process-memory counters are advisory. Count expired-but-not-yet-settled grants conservatively until reconciliation closes them.

## Audit and receipt lifetime

A failed transaction exposes neither a state change nor an audit event/sequence advance. Heartbeats may update contact without individual noisy audit events. Receipts needed for deduplication outlive the retained outcome/range according to source rules. Recovery loops use conditional transitions, so a repeated scan makes no duplicate terminal event.

## Storage enforcement

Use explicit NOT NULL identities, foreign keys, unique receipt keys and enum/positive-range checks. Add partial uniqueness for active ownership/intents where applicable. Enforce immutable payloads, accepted receipts and terminal records through restricted service APIs and migration-level triggers. CHECK expressions must reject missing mandatory data explicitly rather than allowing SQL NULL to pass.
