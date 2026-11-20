# Local access and threat model

This release baseline retains the supplied requirement identifiers and behavioral boundaries. Detailed implementation choices are linked from the component design index.

## Local access and operational limits

Bind to loopback, serve assets locally, check Host/Origin and protect browser mutations against cross-site requests. Fictional producer/operator/consumer identities demonstrate authorization; they are not real account authentication. Generate disposable local session values if required and keep them out of source, URLs and logs.

Reject arbitrary URLs, paths, executable input, duplicate JSON keys, excessive nesting and unknown fields. Fault controls are disabled by default. No real API keys or production connections are required. Treat trusted local processes as the demonstration boundary; general hostile-code sandboxing is deferred.
