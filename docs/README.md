# Documentation guide

Read the product requirements for the human problem, then the architecture for the system boundary. Implementers should read the transaction protocol before the individual components. All requirements describe intended release behavior, not observed results.

| Area | Read |
| --- | --- |
| Product requirements and scope | [Document](requirements/product.md) |
| Software requirements specification | [Document](requirements/software.md) |
| Data model and integrity contract | [Document](contracts/data-model.md) |
| API contract | [Document](contracts/api.md) |
| Browser journeys and accessibility | [Document](requirements/browser-journeys.md) |
| Local access and threat model | [Document](security/access-and-threats.md) |
| Operations and performance plan | [Document](operations/runbook.md) |
| Acceptance and failure-test plan | [Document](testing/acceptance-plan.md) |
| Delivery milestones and definition of done | [Document](delivery/implementation-plan.md) |
| Primary technical references | [Document](architecture/references.md) |
| Architecture | [High-level design](architecture/high-level-design.md), [decisions](architecture/decisions.md) |
| Components | [Low-level design index](design/README.md) |
| Correctness | [Transactions](design/transactions.md), [state machines](design/state-machines.md) |
| Coverage | [Traceability](requirements/traceability.md), [review findings](review/source-review.md) |
| Verification | [Harness and fixtures](testing/harness-and-fixtures.md), [execution report](testing/acceptance-report.md) |
| Handover | [Backup and restore](operations/backup-restore.md), [status](delivery/status.md) |

The original source is retained in [source](source/Reliable_Message_Broker_Requirements.md). Its programme/ownership clauses describe external gates; they do not instruct this repository to submit forms or claim approval. This baseline is AI-assisted and requires participant review where independent authorship is required.

Additional contracts: [quality requirements](requirements/quality.md), [storage and entity relationships](design/storage.md).
