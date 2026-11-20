# Requirement traceability

Every source section is covered below, including requirements stated in prose. Every numbered acceptance case is retained verbatim in the acceptance plan. Coverage describes intended verification, not test success.

| Source section | Requirement/design coverage | Verification |
| --- | --- | --- |
| 1 Purpose and success | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 2 People and use case | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 3 Environment and boundaries | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 4 Core model and guarantees | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 5 Reference demonstration | [Contract](../requirements/product.md) | Acceptance plan and milestone gate |
| 6 Queue and routing requirements | [Contract](../requirements/software.md); [Design](../design/routing.md) | Acceptance plan and milestone gate |
| 7 Publish and idempotency contract | [Contract](../requirements/software.md); [Design](../design/routing.md) | Acceptance plan and milestone gate |
| 8 Delivery state machine | [Contract](../requirements/software.md); [Design](../design/deliveries.md) | Acceptance plan and milestone gate |
| 9 Claim leases and acknowledgements | [Contract](../requirements/software.md); [Design](../design/deliveries.md) | Acceptance plan and milestone gate |
| 10 Retry expiry and dead letters | [Contract](../requirements/software.md); [Design](../design/recovery.md) | Acceptance plan and milestone gate |
| 11 Ordering concurrency and transaction requirements | [Contract](../requirements/software.md); [Design](../design/publish.md) | Acceptance plan and milestone gate |
| 12 Recovery and shutdown | [Contract](../requirements/software.md); [Design](../design/recovery.md) | Acceptance plan and milestone gate |
| 13 Architecture and logical records | [Contract](../contracts/data-model.md) | Acceptance plan and milestone gate |
| 14 API contract | [Contract](../contracts/api.md) | Acceptance plan and milestone gate |
| 15 Browser requirements | [Contract](../requirements/browser-journeys.md) | Acceptance plan and milestone gate |
| 16 Local access and operational limits | [Contract](../security/access-and-threats.md) | Acceptance plan and milestone gate |
| 17 Metrics backup and retained evidence | [Contract](../operations/runbook.md) | Acceptance plan and milestone gate |
| 18 Provisional targets | [Contract](../operations/runbook.md) | Acceptance plan and milestone gate |
| 19 Acceptance catalog | [Contract](../testing/acceptance-plan.md) | Acceptance plan and milestone gate |
| 20 Verification strategy | [Contract](../testing/acceptance-plan.md) | Acceptance plan and milestone gate |
| 21 Session plan | [Contract](../delivery/implementation-plan.md) | Acceptance plan and milestone gate |
| 22 Delivery and open decisions | [Contract](../delivery/implementation-plan.md) | Acceptance plan and milestone gate |
| 23 Primary references | [Contract](../architecture/references.md) | Acceptance plan and milestone gate |

## Numbered functional requirements

| Source ID | Required behavior | Design |
| --- | --- | --- |
| QUE-01 | Create named queues with finite capacity, maximum attempts, retry policy and visibility duration. Reject duplicate names. | [Queue administration and atomic routing](../design/routing.md) |
| QUE-02 | Configure a route as exact-key or fan-out, with versioned bindings to existing queues. A queue receives at most one copy per publish even if several bindings match. | [Queue administration and atomic routing](../design/routing.md) |
| QUE-03 | Route using one committed binding snapshot. Concurrent binding changes cannot partially affect a publish. | [Queue administration and atomic routing](../design/routing.md) |
| QUE-04 | Reject unroutable messages explicitly; do not acknowledge an accidental drop. | [Queue administration and atomic routing](../design/routing.md) |
| QUE-05 | If any destination lacks capacity, reject the entire publish with no queue entries or accepted receipt. Capacity includes delayed and in-flight entries. | [Queue administration and atomic routing](../design/routing.md) |
| QUE-06 | Pause stops new delivery claims, but existing valid deliveries may finish. Publishing to a paused queue remains allowed within limits. | [Queue administration and atomic routing](../design/routing.md) |
| QUE-07 | Drain stops new routed publishes while allowing existing entries to settle. A publish targeting a draining queue fails atomically. | [Queue administration and atomic routing](../design/routing.md) |
| QUE-08 | Initial release has no destructive queue purge. Delete only an empty, unbound queue through an explicit operation; preserve audit history. | [Queue administration and atomic routing](../design/routing.md) |
| PUB-01 | Publish validates producer, route, key, bounded payload and optional TTL before writing. | [Queue administration and atomic routing](../design/routing.md) |
| PUB-02 | A producer supplies a stable request key. Same key and canonical request returns the original receipt; different content returns conflict. | [Queue administration and atomic routing](../design/routing.md) |
| PUB-03 | Payload, destination entries, policy/binding snapshots, durable event and receipt commit atomically before a success response. | [Queue administration and atomic routing](../design/routing.md) |
| PUB-04 | Reject oversized messages before durable mutation. Default maximum payload is 64 KiB; metadata limit is 8 KiB. | [Queue administration and atomic routing](../design/routing.md) |
| PUB-05 | Preserve producer/key receipts while related messages or history are retained. No automatic receipt deletion in the first release. | [Queue administration and atomic routing](../design/routing.md) |
| PUB-06 | An uncertain response is resolved by replaying the same key. A client must not switch keys merely because the first response was lost. | [Queue administration and atomic routing](../design/routing.md) |
| DEL-01 | Claim atomically checks queue state, eligibility, TTL and global/queue/consumer in-flight limits. Process-memory counters are insufficient. | [Claims and delivery authority](../design/deliveries.md) |
| DEL-02 | Claim records consumer instance, generation, attempt number and visibility deadline. Every claim consumes an attempt. | [Claims and delivery authority](../design/deliveries.md) |
| DEL-03 | Default visibility is 15 seconds with extension every 5 seconds; lease expiry is never later than message expiry. | [Claims and delivery authority](../design/deliveries.md) |
| DEL-04 | Extension requires current unexpired authority and cannot revive an expired generation. A fixed maximum processing deadline bounds repeated extension. | [Claims and delivery authority](../design/deliveries.md) |
| DEL-05 | ACK, NACK and extension carry entry ID, delivery ID, consumer ID, generation and operation key. | [Claims and delivery authority](../design/deliveries.md) |
| DEL-06 | Identical accepted ACK retries return the original receipt even after settlement; new stale ACKs cannot affect a newer delivery. | [Claims and delivery authority](../design/deliveries.md) |
| DEL-07 | Consumer registration uses a fresh process-instance ID. A restarted process does not inherit authority merely by reusing a friendly name. | [Claims and delivery authority](../design/deliveries.md) |
| DEL-08 | Backpressure stops issuing work when capacity is full; reads and diagnostics remain available. | [Claims and delivery authority](../design/deliveries.md) |

## Acceptance coverage

| Source case | Durable acceptance oracle | Status |
| --- | --- | --- |
| MB-AC-01 | Create queues/routes; invalid names, missing targets and duplicate bindings are rejected without partial state. | NOT RUN |
| MB-AC-02 | Exact-key routing selects expected queues; fan-out creates one independent entry per destination. | NOT RUN |
| MB-AC-03 | Unroutable publish and full/draining destination reject the whole transaction. | NOT RUN |
| MB-AC-04 | Repeat publish with the same key returns one message; changed content conflicts. | NOT RUN |
| MB-AC-05 | Restart after confirmed publish preserves all entries, payloads and receipts. | NOT RUN |
| MB-AC-06 | Lose response after publish commit; retry returns original destination IDs. | NOT RUN |
| MB-AC-07 | Four consumers contend; no entry has two authorized deliveries; limits hold inside each claim transaction. | NOT RUN |
| MB-AC-08 | Kill consumer after claim; lease expiry redelivers with a newer generation. | NOT RUN |
| MB-AC-09 | Old-generation ACK/extension cannot settle or revive a newer delivery. | NOT RUN |
| MB-AC-10 | Lose ACK response after commit; identical replay returns original ACK receipt. | NOT RUN |
| MB-AC-11 | Fail-first-two fixture succeeds on attempt 3 with preserved retry timestamps. | NOT RUN |
| MB-AC-12 | Permanent rejection and exhaustion atomically preserve one dead letter. | NOT RUN |
| MB-AC-13 | TTL expires while ready, delayed, paused or in flight; no new authority commits at/after deadline. | NOT RUN |
| MB-AC-14 | Lease extension cannot exceed fixed processing deadline or revive expiry. | NOT RUN |
| MB-AC-15 | Pause races claims; drain races publishes; committed order governs outcome. | NOT RUN |
| MB-AC-16 | Binding changes race publish; every receipt identifies one complete routing snapshot. | NOT RUN |
| MB-AC-17 | Duplicate redrive yields one new entry; lineage cap and capacity remain enforced. | NOT RUN |
| MB-AC-18 | Storage failure produces no false confirmation; reconnect resolves uncertain operations by key. | NOT RUN |
| MB-AC-19 | Two recovery loops settle retries/expiry once without duplicate events. | NOT RUN |
| MB-AC-20 | Built-in consumer crashes after effect but before ACK; redelivery does not duplicate its local effect. | NOT RUN |
| MB-AC-21 | Full 10,000-message fixture matches precomputed state counts and result digests after failures. | NOT RUN |
| MB-AC-22 | Oversized payload, path/URL input, unexpected origin and unauthorized mutations are rejected. | NOT RUN |
| MB-AC-23 | Browser reload/two tabs retain accurate states and show stale-control conflicts. | NOT RUN |
| MB-AC-24 | Offline startup, snapshot restore and same-key replay preserve retained outcomes. | NOT RUN |
