# Worked examples

These are small fictional request and protocol examples, not the canonical stress fixtures or recorded test results. Each JSON file has a corresponding schema. Digest-bearing decision inputs must use actual canonical digests fetched from the current reviewed state; no invented digest is supplied here.

| Example | Schema | What it demonstrates |
| --- | --- | --- |
| [document-queue.json](document-queue.json) | [queue-request](../schemas/queue-request.schema.json) | Capacity includes ready, delayed and in-flight entries; maxInFlight is a separate admission limit. |
| [direct-route.json](direct-route.json) | [route-request](../schemas/route-request.schema.json) | Configure after the destination queue exists. Bindings use queue IDs; this example uses readable synthetic IDs. |
| [fan-out-route.json](fan-out-route.json) | [route-request](../schemas/route-request.schema.json) | Create both queues first. Publishing is atomic across the distinct destination queues. |
| [publish-document.json](publish-document.json) | [publish-request](../schemas/publish-request.schema.json) | Send with Idempotency-Key: demo-publish-001. A retry must preserve that key and body. |
| [ack-delivery.json](ack-delivery.json) | [delivery-action](../schemas/delivery-action.schema.json) | Illustrative token fields only: use the IDs/generation actually returned by claim. |
| [retryable-nack.json](retryable-nack.json) | [delivery-action](../schemas/delivery-action.schema.json) | Server policy classifies allowed transient codes; the caller cannot extend its attempt budget. |
| [redrive-request.json](redrive-request.json) | [redrive-request](../schemas/redrive-request.schema.json) | Creates one fresh entry with preserved lineage. The old entry stays terminal. |
| [consumer-registration.json](consumer-registration.json) | [consumer-registration](../schemas/consumer-registration.schema.json) | Use a new consumer instance ID on every process restart. |

Submit mutations with a stable Idempotency-Key and the appropriate fictional role/registered process. Retry the same intent with the same key and body after an uncertain response. A new edited intent needs a new key. An accepted response is required before the browser displays success.

Actual generated large fixtures, independent expected-output hashes and crash evidence are future implementation deliverables. These files claim no measured results.
