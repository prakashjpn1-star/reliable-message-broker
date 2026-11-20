# Acceptance and failure-test plan

Acceptance scenarios check committed records at controlled failure boundaries. Every case remains NOT RUN until implementation evidence is recorded.

## Acceptance catalog

All cases begin NOT RUN. Each records setup, seed, commit, environment, action/barrier, durable before/after state, receipts, timings and invariant results.

| ID | Scenario and pass condition |
| --- | --- |
| MB-AC-01 | Create queues/routes; invalid names, missing targets and duplicate bindings are rejected without partial state. |
| MB-AC-02 | Exact-key routing selects expected queues; fan-out creates one independent entry per destination. |
| MB-AC-03 | Unroutable publish and full/draining destination reject the whole transaction. |
| MB-AC-04 | Repeat publish with the same key returns one message; changed content conflicts. |
| MB-AC-05 | Restart after confirmed publish preserves all entries, payloads and receipts. |
| MB-AC-06 | Lose response after publish commit; retry returns original destination IDs. |
| MB-AC-07 | Four consumers contend; no entry has two authorized deliveries; limits hold inside each claim transaction. |
| MB-AC-08 | Kill consumer after claim; lease expiry redelivers with a newer generation. |
| MB-AC-09 | Old-generation ACK/extension cannot settle or revive a newer delivery. |
| MB-AC-10 | Lose ACK response after commit; identical replay returns original ACK receipt. |
| MB-AC-11 | Fail-first-two fixture succeeds on attempt 3 with preserved retry timestamps. |
| MB-AC-12 | Permanent rejection and exhaustion atomically preserve one dead letter. |
| MB-AC-13 | TTL expires while ready, delayed, paused or in flight; no new authority commits at/after deadline. |
| MB-AC-14 | Lease extension cannot exceed fixed processing deadline or revive expiry. |
| MB-AC-15 | Pause races claims; drain races publishes; committed order governs outcome. |
| MB-AC-16 | Binding changes race publish; every receipt identifies one complete routing snapshot. |
| MB-AC-17 | Duplicate redrive yields one new entry; lineage cap and capacity remain enforced. |
| MB-AC-18 | Storage failure produces no false confirmation; reconnect resolves uncertain operations by key. |
| MB-AC-19 | Two recovery loops settle retries/expiry once without duplicate events. |
| MB-AC-20 | Built-in consumer crashes after effect but before ACK; redelivery does not duplicate its local effect. |
| MB-AC-21 | Full 10,000-message fixture matches precomputed state counts and result digests after failures. |
| MB-AC-22 | Oversized payload, path/URL input, unexpected origin and unauthorized mutations are rejected. |
| MB-AC-23 | Browser reload/two tabs retain accurate states and show stale-control conflicts. |
| MB-AC-24 | Offline startup, snapshot restore and same-key replay preserve retained outcomes. |

## Verification strategy

Unit tests cover routing, backoff, TTL arithmetic, digests and legal state transitions. Integration tests exercise actual storage constraints and transaction rollback. Concurrent-process tests use real local producers/consumers. Browser tests cover publish, pause, inspect, redrive and stale tabs.

Use deterministic barriers before/after publish commit, after delivery grant, after consumer effect before ACK, before/after ACK commit, and around expiry versus completion. Repeat each crash window at least 20 times and contention/stale-authority cases at least 100 rounds. Preserve failing seeds and rounds. Run a 30-minute bounded soak.

Invariants: every confirmed publish has all destination entries; no orphan payload/entry; one authorized generation per entry; terminal entries never reopen; capacity bounds hold; accepted changes have audit events; dead-letter/redrive lineage remains traceable; stale authority cannot ACK; messages remain accounted for. Protocol-level redelivery is allowed and must be visible, not hidden by an incorrect no-duplicates claim.
