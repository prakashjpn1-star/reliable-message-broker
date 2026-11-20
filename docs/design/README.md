# Low-level component designs

Every component follows the same [transaction rules](transactions.md), [API errors](../contracts/api.md) and [data constraints](../contracts/data-model.md).

- [Queue administration and atomic routing](routing.md)
- [Producer validation and receipts](publish.md)
- [Claims and delivery authority](deliveries.md)
- [ACK NACK and consumer effects](acknowledgements.md)
- [Retry TTL expiry and recovery](recovery.md)
- [Dead letters and controlled redrive](dead-letters.md)
- [State machines](state-machines.md)
- [Browser and control API](browser-and-api.md)
- [Observability](observability.md)

Each design includes an algorithm, diagram, atomicity/recovery behavior and applicable source contracts. The [test harness](../testing/harness-and-fixtures.md) defines evidence required to prove the design.

- [Storage and entity relationships](storage.md)
