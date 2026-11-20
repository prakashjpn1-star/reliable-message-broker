# Claims and delivery authority

Default visibility is 15 seconds, extension interval 5 seconds and fixed maximum processing duration 60 seconds. One partial unique active-delivery index protects each entry. Count still-open expired grants conservatively until reconciliation closes them.

```mermaid
flowchart TD
    Candidate["READY entry"] --> Guards["Queue, TTL and shared limits"] --> Token["New delivery generation"] --> Claim["Commit attempt and receipt"]
```

## Operation algorithm

1. Begin mutation before reading candidate eligibility or shared occupancy.
2. Check queue pause, READY state, TTL and global/queue/consumer limits using durable rows.
3. Increment generation and attempt number; create delivery and lease bounded by TTL and fixed processing deadline.
4. Commit claim and keyed receipt, then return authority to the registered consumer instance.

## Inputs outputs and ownership

The service receives a validated domain request, caller/process identity, target identity, expected revision or epoch where applicable, and stable operation key. Read authoritative mutable state inside the accepting transaction. Return the committed receipt/current bounded result, or a stable error without implying acceptance. Interfaces and fields are specified in the [API](../contracts/api.md) and [data model](../contracts/data-model.md).

## Atomicity and failure handling

Publish locks one routing snapshot and all destination admission decisions in one canonical transaction. Claim protects shared capacity and authority together. ACK/NACK/expiry/redrive each commits its event and receipt with the state transition. Consumer effects use a separate store and deduplicate by queue-entry identity.

Use conditional source-state checks, uniqueness constraints and atomic audit/receipt writes. On contention roll back and retry the entire operation under bounded backoff, preserving the key. On uncertain response consult the receipt before repeating work. A stale revision/epoch requires refresh or fresh authority, never a silently rewritten request. Database unavailability stops new confirmations.

## Required behavior

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

## Ordering concurrency and transaction requirements

READY selection uses eligible timestamp then stable entry ID. Parallel consumers, retries and requeues can change completion order; strict FIFO processing is not promised. No second delivery is authorized until the prior generation has expired or been closed.

All mutations use one documented transactional protocol: check receipt, check queue/control revision and durable time, validate authority/capacity, mutate entry/delivery, write effect/event, persist receipt, then commit. Locks or serializable conditional updates must protect both candidate rows and shared capacity. Queue pause, expiry, ACK and claim races have a deterministic committed winner.

No handler computation or network I/O occurs under a storage write lock. Retry whole transactions on documented transient conflicts using bounded backoff and the same key. If commit outcome is uncertain, query/replay the durable receipt. Storage failure never produces a fabricated confirmation.

## Verification boundary

Exercise each applicable acceptance row linked in [traceability](../requirements/traceability.md). Test both serialized race orders, rollback before commit and response loss after commit. Assertions inspect committed state and immutable receipts; browser messages and process logs alone are insufficient evidence.
