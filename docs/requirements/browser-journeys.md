# Browser journeys and accessibility

The browser shows confirmed server state, pending actions and actionable errors. Reloading or opening another tab does not change durable progress.

## Browser requirements

Screens: queue dashboard; route/binding editor; synthetic publish form; queue-entry detail; delivery/consumer view; retry timeline; dead-letter inbox with redrive review; readiness/settings/metrics page. Every screen needs loading, empty, stale, error and disconnected states.

Show ready/delayed/in-flight/ACKed/dead-lettered/expired counts with text as well as colour. Explain that pausing does not stop already granted work and ACK does not prove exactly-once external processing. Refresh must not alter delivery. Two tabs use server revisions and show conflicts. Keyboard access and an accessible table must cover every diagram action. Confirm redrive and queue deletion; never imply terminal history was erased.
