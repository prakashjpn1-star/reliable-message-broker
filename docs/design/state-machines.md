# State machines and control races

```mermaid
stateDiagram-v2
    [*] --> READY
    READY --> IN_FLIGHT: bounded claim
    IN_FLIGHT --> ACKED: valid ACK
    IN_FLIGHT --> RETRY_WAIT: retryable failure or lease loss
    RETRY_WAIT --> READY: stored due time reached
    IN_FLIGHT --> DEAD_LETTERED: permanent error or exhaustion
    READY --> EXPIRED: TTL reached
    RETRY_WAIT --> EXPIRED: TTL reached
    IN_FLIGHT --> EXPIRED: revoke authority at TTL
    ACKED --> [*]
    DEAD_LETTERED --> [*]
    EXPIRED --> [*]
```

Terminal outcomes do not reopen. Idempotent replay returns an existing receipt; it does not repeat a transition. Controls use expected revisions, persisted state and the transaction protocol.

A message can have independent entries in several queues. A publish receipt proves atomic routing, while an ACK settles one queue entry. Consumer execution can repeat; external exactly-once processing is outside scope.

## Race matrix

| Race | Required result |
| --- | --- |
| New mutation versus matching retry | Return one original outcome; changed content conflicts |
| Current owner versus replacement | Epoch/generation determines authority; stale update rejected |
| Control versus new work | First valid serialized transition determines admission |
| Completion versus deadline | Check durable time inside writer; expired authority cannot make a new commit |
| Read versus retention/snapshot transition | Return one consistent state and bounds or an explicit error |
| Commit versus lost response | Replay receipt; do not infer rollback from transport failure |

The component documents supply additional source state values and exact transitions. Each race must be tested in both orders using deterministic barriers.
