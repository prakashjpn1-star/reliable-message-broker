# Architecture decisions

## ADR 1 Local Python services and one database technology

The reference design uses Python for the services, SQLite for transactional storage, and locally bundled HTML, CSS and JavaScript for the browser. Services communicate through loopback HTTP APIs. Separate logical stores use the same database technology without assuming a transaction across them.

The HTTP framework, validation/canonicalization libraries and exact dependency versions remain open pending the permitted environment inventory. The dependency record will include version pins, hashes, licenses and offline availability. This is a design choice; it is not a claim that those dependencies have already been verified.

## ADR 2 Serialize canonical mutations

Use explicit BEGIN IMMEDIATE transactions, WAL, foreign keys enabled on every connection, FULL synchronous durability and supported local filesystems. Keep computation/network/human decisions outside write transactions. This is a proposed correctness strategy, subject to contention/performance proof. SQLite documents [transaction acquisition](https://www.sqlite.org/lang_transaction.html) and [WAL deployment limits](https://www.sqlite.org/wal.html).

## ADR 3 One canonical digest contract

Propose RFC 8785 canonical JSON with SHA-256, verified against standard vectors. Reject duplicate keys, invalid Unicode and nonfinite numbers before hashing. Exact decimal business values use declared strings. Request digests include operation target/generation and ordered content; receipt keys omit retry number. Exact packages remain a pre-implementation gate.

## ADR 4 Durable state plus atomic audit

Maintain indexed current state and append audit in the same mutation transaction. Allocate sequence from the owning stream/queue/hub under the write lock. Wall-clock time is diagnostic, never an ordering key. Receipts preserve the original committed answer even when current state changes.

## ADR 5 Browser polling and accessible equivalents

Propose one-second healthy polling, bounded pagination, server revision checks and a complete table alternative to every graph. A pending button is not a committed result. Keep a mutation key across uncertain transport and reload; a changed user intent gets a new operation.

## ADR 6 Conservative retention and restore

Publish locks one routing snapshot and all destination admission decisions in one canonical transaction. Claim protects shared capacity and authority together. ACK/NACK/expiry/redrive each commits its event and receipt with the state transition. Consumer effects use a separate store and deduplicate by queue-entry identity.

Use consistent database snapshots, retain required receipts/history, and restore into isolated paths with fresh process identities. Never silently reset state to fix an error. A backup restores only the commits it actually contains.

## Decisions still open

Exact framework/dependency versions; target desktop and filesystem; canonical serialization implementation; configurable storage/response caps; complete disposable role/session bootstrap; fixture digests; measured targets. Resolve these before claiming implementation readiness.
