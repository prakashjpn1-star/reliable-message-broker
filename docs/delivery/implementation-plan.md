# Delivery milestones and definition of done

Delivery proceeds through runnable milestones with explicit evidence and release gates. Each milestone builds on the guarantees established by the previous one.

## Session plan

| Milestone | Work and checkable exit |
| --- | --- |
| 1 | Choose verified tools/storage; local browser, readiness and schema migration work. |
| 2 | Implement queues and route validation; exact and fan-out fixtures pass. |
| 3 | Implement atomic publish/receipts; lost-response tests pass. |
| 4 | Implement delivery grants and in-flight limits; competing consumers pass. |
| 5 | Add leases/fencing/ACK/NACK; hard-kill and stale-token tests pass. |
| 6 | Add backoff, TTL and recovery loop; restart preserves due times. |
| 7 | Add dead letters/redrive and admission controls; race tests pass. |
| 8 | Complete browser investigation and local authorization flows. |
| 9 | Execute fixture, load, soak, backup and offline acceptance; record limitations. |

Each milestone may span several sessions. End with runnable behaviour, relevant tests, a commit/checkpoint and continuation notes. Do not substitute documentation checks for runtime acceptance.

## Delivery and open decisions

Deliver source/licence notices, local startup scripts, schema migrations, API contracts, fixture generator/expected outputs, browser application, consumer examples, crash harness, acceptance report, measurements, consistent backup/restore instructions and session history.

The reference architecture uses Python services, SQLite storage and canonical JSON request digests. Exact framework/library versions and configured capacity/TTL limits remain tracked decisions in the architecture record. Release requires every mandatory acceptance case and repetition gate to pass offline, retained evidence for crashes, and no unresolved defect invalidating the stated guarantees. No programme approval is implied by a completed specification.
