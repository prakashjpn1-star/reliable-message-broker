# Software requirements specification

A message can have independent entries in several queues. A publish receipt proves atomic routing, while an ACK settles one queue entry. Consumer execution can repeat; external exactly-once processing is outside scope.

## Queue and routing requirements

| ID | Requirement |
| --- | --- |
| QUE-01 | Create named queues with finite capacity, maximum attempts, retry policy and visibility duration. Reject duplicate names. |
| QUE-02 | Configure a route as exact-key or fan-out, with versioned bindings to existing queues. A queue receives at most one copy per publish even if several bindings match. |
| QUE-03 | Route using one committed binding snapshot. Concurrent binding changes cannot partially affect a publish. |
| QUE-04 | Reject unroutable messages explicitly; do not acknowledge an accidental drop. |
| QUE-05 | If any destination lacks capacity, reject the entire publish with no queue entries or accepted receipt. Capacity includes delayed and in-flight entries. |
| QUE-06 | Pause stops new delivery claims, but existing valid deliveries may finish. Publishing to a paused queue remains allowed within limits. |
| QUE-07 | Drain stops new routed publishes while allowing existing entries to settle. A publish targeting a draining queue fails atomically. |
| QUE-08 | Initial release has no destructive queue purge. Delete only an empty, unbound queue through an explicit operation; preserve audit history. |

Mutable queue settings use expected revisions. A change must not reduce capacity below current occupancy. Retry/lease policy is snapshotted into a queue entry at routing; changing queue defaults affects new entries. Pause/drain are current admission controls, not entry policies.

## Publish and idempotency contract

| ID | Requirement |
| --- | --- |
| PUB-01 | Publish validates producer, route, key, bounded payload and optional TTL before writing. |
| PUB-02 | A producer supplies a stable request key. Same key and canonical request returns the original receipt; different content returns conflict. |
| PUB-03 | Payload, destination entries, policy/binding snapshots, durable event and receipt commit atomically before a success response. |
| PUB-04 | Reject oversized messages before durable mutation. Default maximum payload is 64 KiB; metadata limit is 8 KiB. |
| PUB-05 | Preserve producer/key receipts while related messages or history are retained. No automatic receipt deletion in the first release. |
| PUB-06 | An uncertain response is resolved by replaying the same key. A client must not switch keys merely because the first response was lost. |

Use a documented deterministic serialization for request digests, reject duplicate JSON keys/nonfinite numbers, and define whether payload bytes are raw or canonical JSON. The digest covers route, routing key, payload, metadata and TTL. A receipt includes message ID, queue-entry IDs, binding version and accepted time.

## Delivery state machine

Queue-entry states are READY, IN_FLIGHT, RETRY_WAIT, ACKED, DEAD_LETTERED and EXPIRED. ACKED, DEAD_LETTERED and EXPIRED are terminal for that entry. Message-level views summarize all entry outcomes; they do not collapse independent queues into one ACK.

| Transition | Guard and atomic outcome |
| --- | --- |
| READY to IN_FLIGHT | Queue unpaused, entry unexpired, consumer compatible and capacity available; increment generation and create delivery. |
| IN_FLIGHT to ACKED | Current consumer/token, unexpired lease and message TTL; store ACK receipt and close delivery. |
| IN_FLIGHT to RETRY_WAIT | Retryable NACK or expired lease with attempts remaining; persist next eligibility once. |
| RETRY_WAIT to READY | Due timestamp reached and TTL still valid; pause can leave it ready but unclaimable. |
| IN_FLIGHT to DEAD_LETTERED | Permanent rejection or exhausted attempt budget; preserve terminal envelope. |
| Nonterminal to EXPIRED | TTL reached; revoke any current delivery and preserve expiry reason. |

Expired delivery attempts remain in history as LOST; explicit failed attempts record NACK; acknowledged attempts record ACKED. At most one authorized in-flight generation exists per queue entry.

## Claim leases and acknowledgements

| ID | Requirement |
| --- | --- |
| DEL-01 | Claim atomically checks queue state, eligibility, TTL and global/queue/consumer in-flight limits. Process-memory counters are insufficient. |
| DEL-02 | Claim records consumer instance, generation, attempt number and visibility deadline. Every claim consumes an attempt. |
| DEL-03 | Default visibility is 15 seconds with extension every 5 seconds; lease expiry is never later than message expiry. |
| DEL-04 | Extension requires current unexpired authority and cannot revive an expired generation. A fixed maximum processing deadline bounds repeated extension. |
| DEL-05 | ACK, NACK and extension carry entry ID, delivery ID, consumer ID, generation and operation key. |
| DEL-06 | Identical accepted ACK retries return the original receipt even after settlement; new stale ACKs cannot affect a newer delivery. |
| DEL-07 | Consumer registration uses a fresh process-instance ID. A restarted process does not inherit authority merely by reusing a friendly name. |
| DEL-08 | Backpressure stops issuing work when capacity is full; reads and diagnostics remain available. |

The default maximum processing duration is 60 seconds per delivery, independent of extensions. Time is sampled from the durable store's chosen wall-clock authority; exact equality to a deadline is expired. Recheck authority at the serialized mutation point. Record clock-source and clock-jump limitations.

## Retry expiry and dead letters

Backoff after attempt n is min(maxDelay, initialDelay × multiplier^(n−1)). Defaults: 3 attempts including the initial claim, initial delay 1 second, multiplier 2, maximum 30 seconds. Persist the calculated eligibility time; restart must not recalculate it. Optional jitter is disabled in deterministic tests.

TTL is measured from publish acceptance and continues during queue pause, retry and broker downtime. Absence of TTL means no time-based expiry, subject to storage limits and explicit retention. TTL expiry takes precedence over a new claim/ACK once its deadline is reached. Retried ACK receipts prove an old commit without reauthorizing work.

Dead-letter transfer is atomic with source-entry terminal settlement and stores a reference to immutable payload, failure reason, root message, queue, attempts and timestamps. It is an internal durable record, not an external publish that can disappear between two operations.

Redrive requires operator role, expected dead-letter revision and request key. It creates a new entry with a new ID, preserved root lineage and a fresh snapshotted policy; the old entry remains terminal. Repeated redrive requests return one new entry. A dead letter permits one accepted redrive; subsequent cycles create new dead letters. Enforce a configurable lineage redrive cap, default 3, and queue capacity. Redrive of expired input requires explicit fresh TTL selection.

## Ordering concurrency and transaction requirements

READY selection uses eligible timestamp then stable entry ID. Parallel consumers, retries and requeues can change completion order; strict FIFO processing is not promised. No second delivery is authorized until the prior generation has expired or been closed.

All mutations use one documented transactional protocol: check receipt, check queue/control revision and durable time, validate authority/capacity, mutate entry/delivery, write effect/event, persist receipt, then commit. Locks or serializable conditional updates must protect both candidate rows and shared capacity. Queue pause, expiry, ACK and claim races have a deterministic committed winner.

No handler computation or network I/O occurs under a storage write lock. Retry whole transactions on documented transient conflicts using bounded backoff and the same key. If commit outcome is uncertain, query/replay the durable receipt. Storage failure never produces a fabricated confirmation.

## Recovery and shutdown

At startup verify schema and storage health before accepting mutations. Reconcile expired delivery leases, due retries and TTL expiry in bounded batches. Multiple local recovery loops must not create duplicate transitions. Do not reset persisted timestamps.

Graceful consumer shutdown stops claims and finishes or explicitly releases current work. Hard stop relies on lease expiry. Broker restart retains all confirmed publishes, current entry states and receipts. Consumers reconnect using fresh instances and reconcile uncertain ACKs with their original keys.

If storage is unavailable, stop confirmations and new grants. Return an explicit unavailable error. After reconnect, revalidate authority before accepting any result. Full-disk and transaction-conflict behaviour must be exercised without risking unrelated workspace files.
