# Schemas and reference storage model

These JSON Schemas use draft 2020-12 and describe proposed version 1 contracts. They complement the [API contract](../docs/contracts/api.md); the runtime is not implemented yet. The field limits newly made concrete here are proposal defaults, not measured limits or silent changes to the source scope.

| Schema | Purpose |
| --- | --- |
| [publish-request.schema.json](publish-request.schema.json) | Durable publish request |
| [queue-request.schema.json](queue-request.schema.json) | Queue creation and default policy |
| [route-request.schema.json](route-request.schema.json) | Versioned exact or fan-out route |
| [delivery-action.schema.json](delivery-action.schema.json) | Fenced delivery action envelope |
| [redrive-request.schema.json](redrive-request.schema.json) | Explicit dead-letter redrive request |
| [consumer-registration.schema.json](consumer-registration.schema.json) | Fresh consumer instance registration |

[reference-schema.sql](reference-schema.sql) supplies relational keys, foreign keys, enum constraints and indexes. Sections labeled as separate stores belong in separate databases. This is a reference design, not an installed migration. Migration checksums, immutable-field triggers, role checks, byte limits and the complete transaction protocol remain implementation work.

JSON Schema checks structure, not database state. It cannot prove a current revision, consecutive feed/offset range, valid lease, safe retention cutoff or sufficient capacity. Enforce those rules in the accepting transaction. Encoded byte limits, duplicate-key rejection, Unicode/canonicalization and SHA-256 verification also require the parser/domain implementation.

Idempotency-Key, correlation IDs and demo identity normally travel in headers/session context and are not duplicated in these body schemas. Path IDs must agree with any body identity. Examples use fictional readable IDs; resolve names to registered IDs before actual submission.

The publish schema specializes payload to the allowlisted document fixture handler. Add versioned payload schemas for any future permitted handler. Queue policy examples use a proposed 1–10 attempt bound. The delivery-action envelope selects an endpoint; strip action when sending the endpoint-specific body.

No schema parser or SQL setup result is a runtime durability acceptance result.
