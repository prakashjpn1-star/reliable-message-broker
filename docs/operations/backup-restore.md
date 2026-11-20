# Backup restore and safe local setup

Services use ordinary local processes and locally served assets. Verify the allowed interpreter, database build, package mirror and offline test tools before pinning dependencies. Runtime commands/scripts are implementation deliverables, not existing commands in this documentation repository. Keep live state on supported local disk outside OneDrive/network shares.

```mermaid
flowchart LR
    Stop["Stop admission and owning processes"] --> Snapshot["Consistent store snapshots and identities"]
    Snapshot --> Verify["Integrity, schema and digest verification"]
    Verify --> Restore["Restore into isolated local directories"]
    Restore --> Recover["Fresh processes and durable reconciliation"]
```

Version migrations explicitly, record checksums and refuse newer/incompatible schemas. Never auto-reset. Back up all required stores, receipts, identities, policies, history and positions together. For this multi-process demonstration, quiesce and stop the relevant services to establish a coordinated cold boundary; use the chosen database backup mechanism rather than separately timed copies of live main files. Preserve WAL-visible commits.

Broker and consumer-effect stores form the end-to-end restore set. Preserve routing snapshots, entry generations, dead letters, lineage and publish/ACK receipts. Restart consumers with fresh instance IDs.

Restore into new validated paths with the original data untouched, verify integrity/foreign keys/schema and digests, then start services with fresh process identities. Reconcile persisted due times rather than resetting them. Retry known keys and assert preserved outcome identities. A backup cannot recover commits made after it.

Graceful shutdown stops new work and permits bounded completion; hard-stop recovery uses durable state and authority expiry. In an outage stop confirming work, preserve evidence, restore storage access, recheck readiness and revalidate authority before continuing. Reset-test-data commands must refuse normal runtime paths.
