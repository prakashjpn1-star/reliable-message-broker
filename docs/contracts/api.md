# API contract

This release baseline retains the supplied requirement identifiers and behavioral boundaries. Detailed implementation choices are linked from the component design index.

## API contract

Suggested endpoints; equivalent documented contracts are acceptable:

| Method and path | Behaviour |
| --- | --- |
| GET /api/health and /api/ready | Separate liveness from storage/schema readiness. |
| POST /api/queues | Create bounded queue. |
| PUT /api/queues/{id} | Revise defaults using expected revision. |
| POST /api/queues/{id}/pause, /resume, /drain | Idempotent admission control. |
| POST /api/routes and PUT /api/routes/{id}/bindings | Create/update versioned routing. |
| POST /api/messages | Keyed durable publish. |
| POST /internal/consumers/register | Register disposable local consumer instance. |
| POST /internal/queues/{id}/claim | Claim bounded entries; empty response is explicit. |
| POST /internal/deliveries/{id}/extend, /ack, /nack | Fenced delivery operations with request keys. |
| GET /api/queues/{id}/entries and /events | Filtered, paginated inspection. |
| GET /api/messages/{id} | Payload metadata and independent routing outcomes. |
| GET /api/dead-letters | Filter failures and lineage. |
| POST /api/dead-letters/{id}/redrive | Create one keyed new delivery lineage entry. |
| GET /api/consumers and /api/metrics | Capacity, heartbeat age, rates and backlog. |

Example publish body: `{"producerId":"demo-producer","route":"documents","routingKey":"transform","payload":{"fixtureId":"doc-001"},"ttlSeconds":300}`. Idempotency key is an operation-scoped header. Claim returns IDs, immutable payload reference, generation, expiry and processing deadline.

Use 400 for malformed JSON, 422 for invalid contracts, 403 for denied roles, 404 for missing resources, 409 for stale authority/revision/key conflicts, 413 for oversized input and 503 for unavailable storage. Errors include code/message/correlation ID and bounded details. Lists default to 50, maximum 200, with stable cursor ordering. Empty polls use a fresh key after a known response; transport retries preserve the previous key.
