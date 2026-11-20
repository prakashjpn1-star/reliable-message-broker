# Quality requirements and measurable targets

Correctness comes first: no fabricated confirmation, no stale authority mutation, and no loss of accepted history. Performance figures remain provisional until measured on the supplied desktop.


Targets must be measured on the supplied desktop and revised openly if needed: 100 queues; 100,000 retained entries; 4 producers/4 consumers; 64 KiB payload limit; 100 small-message publishes per second for 60 seconds; p95 available-capacity claim within 2 seconds; expiry/recovery within 5 seconds after due time when services are healthy; p95 paginated read under 500 ms; browser update within 3 seconds; services/store under 2 GiB steady-state RAM excluding browser/build tools. Throughput tests exclude deliberate slow handlers. These are not existing results.



## Shared release criteria

The complete journey works offline with loopback available, ordinary local processes and synthetic data. Browser state uses confirmed server revisions, text labels and keyboard-accessible equivalents. Inputs, pages, logs and artifact/record sizes are bounded before expensive work.

Measure p50/p95/max where relevant, record warm-up/sample count and retain raw failures. Every report includes build/commit, CPU/RAM/OS, store/interpreter/package versions, durability settings, worker counts, payload sizes, fixture seed and exact method. Do not silently relax a target or remove a slow sample.

Run crash windows at least 20 times, contention/authority races at least 100 rounds, and a bounded 30-minute soak. Report NOT RUN until evidence exists. Actual reports are separate from regenerable templates.
