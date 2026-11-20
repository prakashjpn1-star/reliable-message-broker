# Data model and integrity contract

This release baseline retains the supplied requirement identifiers and behavioral boundaries. Detailed implementation choices are linked from the component design index.

## Architecture and logical records

Logical components: same-origin browser/API; routing and administration service; transactional message/queue store; delivery service; retry/expiry reconciler; independent local producers/consumers; fault harness. One broker service may combine these responsibilities initially. Multiple brokers or a consensus layer are not required.

| Entity | Essential fields and constraints |
| --- | --- |
| Queue | Nonnull ID/name, state/revision, capacity and default policy. |
| Route and binding version | Mode, exact key where relevant, immutable binding snapshot/digest. |
| Message | Producer/key, immutable payload/digest, accepted time, optional expiry. |
| QueueEntry | Unique message/queue pair for publish routing, origin/redrive lineage, policy snapshot and current state. |
| Delivery | Entry, consumer, attempt/generation, lease, processing deadline, outcome; one active authority per entry. |
| DeadLetter | Terminal entry reference, reason, preserved history and redrive receipt/reference. |
| Consumer | Instance ID, permitted queues, capacity and last contact. |
| Receipt | Unique actor/operation/target/key, request digest and committed response. |
| Event | Unique queue/sequence, entity/action/actor, bounded payload and timestamp. |
| ConsumerEffect | Demonstration-only deduplication by queue-entry ID, result and digest. |

Use explicit nonnull identity constraints, foreign keys, positive limits and enumerated states. Nullable expressions must not accidentally pass mandatory checks. Index ready entries, due retries/expiry, active deliveries and scoped receipts. Allocate audit sequence in the same transaction as the transition. Immutable payloads and terminal outcomes require service guards and appropriate storage enforcement.
