# Source review and resolved design questions

The main design risks concern the boundary between a committed outcome and a lost response. The following decisions address those risks; runtime acceptance evidence is still pending.

| Question or risk | Design response |
| --- | --- |
| Queue-level ACK must not collapse independent fan-out outcomes. | Each destination has its own queue entry and ACK outcome; the message view summarizes those independent results. |
| Capacity checks outside the publish transaction can overfill a destination. | Publish checks one routing snapshot and every destination’s occupancy inside the same transaction as entry creation. |
| A lost ACK response must replay proof rather than revive a stale delivery. | An identical committed ACK receipt is replayable after expiry; every new ACK still requires the current valid delivery token. |
| Redrive must be a new lineage entry; reopening a terminal entry erases history. | Redrive creates a fresh entry with lineage and a receipt. The source entry stays terminal. |
| Dependency compatibility | Python and SQLite are the reference stack. Exact framework/library versions depend on the permitted environment inventory. |
| Expected hashes not yet generated | Require deterministic generator and independent oracle; keep actual result fields unfilled. |
| Durability confused with exactly-once effects | State acceptance, delivery and local-effect guarantees separately. |
| Generated evidence overwritten | Keep result reports separate; no build script regenerates executed acceptance outcomes. |

New design choices are labeled proposed: explicit SQLite writer serialization, canonical JSON library, one-second browser polling and conservative recovery. Open items include exact versions, configured hard caps, workload measurements and immutable-field migration enforcement.

The original requirements remain archived for traceability. Programme eligibility and participant authorship are assessed separately from these technical design decisions.
