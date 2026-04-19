## Logging & Observability:

- Integrate structured logging using the `tracing` or `log` crate for key business logic transitions.

- Use appropriate log levels: `error!` for failures, `warn!` for unexpected but non-fatal events, and `info!` for high-level flow tracking.

- Ensure log messages provide enough context (e.g., specific IDs or state changes) to assist in debugging without being overly verbose.

> **The Golden Rule of Logging:** Logs are written by software, parsed by machines, and only read by humans _when things go wrong_. Optimize for the machine first, human second.

### Mandate Structured Logging

Plain text logs (`"User 1234 failed to checkout"`) are dead. Mandate JSON-formatted structured logging. This allows log aggregators (ELK, Datadog, Splunk) to index fields and generate metrics.

- **Bad:** `[INFO] Order 9912 processed for user 55.`
- **Good:** `{"level": "INFO", "message": "Order processed", "order_id": 9912, "user_id": 55, "latency_ms": 142}`

### Strict Log Level Discipline

Developers often abuse log levels. Define exactly what each means so your alerting doesn't suffer from alert fatigue.

- **ERROR:** The system is broken, a transaction failed, or data is lost. _Rule: Every ERROR log should trigger an alert or wake someone up._
- **WARN:** An unexpected event occurred, but the system recovered (e.g., a 3rd party API timed out, but the fallback cache worked).
- **INFO:** Significant state changes or business events (e.g., "Service started", "Payment processed").
- **DEBUG:** Diagnostic info useful for developers troubleshooting. (Usually disabled in production).
- **TRACE:** Extremely verbose step-by-step execution details. (Rarely used outside local development).

#### The DEBUG Standard

DEBUG logs tell you exactly how something happened. Because they are incredibly noisy and expensive (both in storage costs and application performance), they require strict guardrails:

1. **Targeted Toggling**: Do not enable `DEBUG` globally in production. The system must support enabling `DEBUG` dynamically at runtime for a specific scope (e.g., via an `X-Force-Debug: true` header or a feature flag tied to a specific `user_id`), without requiring a service restart.
2. **State Snapshots, Not Breadcrumbs**: Don't log `"Entering calculateTax()"`. Capture the state of the machine going into, and coming out of, complex logic. Log the attempt number, backoff metrics, and gateway name.
3. **Guard Expensive Serializations**: JSON serialization burns CPU cycles. Wrap expensive debug payloads in conditional checks so the app doesn't build massive strings in memory when running at the `INFO` level. (e.g., `if (logger.isDebugEnabled()) { ... }`).
4. **Truncate Massive Payloads**: Cap logged request/response bodies to a reasonable limit (e.g., 2KB - 5KB). Logging a 5MB base64 profile picture payload will choke our log forwarders.

### Context and Traceability (Correlation IDs)

An isolated log entry is almost useless in a microservices architecture.

- Every incoming request must be assigned a unique **Correlation ID** at the gateway.
- This ID must be passed to every downstream service and included in _every_ log payload.
- This allows QA and Eng to trace a single user's journey across 15 different microservices with one search query.

### Zero Tolerance for PII and Secrets

This is a critical QA and security gate. Your principles must explicitly forbid logging:

- Passwords, API keys, or bearer tokens.
- Personally Identifiable Information (PII) like SSNs, full credit card numbers, or unhashed email addresses.
- _Implementation Tip:_ Mandate the use of a logging middleware that automatically sanitizes or masks known sensitive keys (e.g., `"password": "***"`).

### Actionable Error Messages

"Failed to process" is a useless log. Good logs answer three questions: **What failed? Why did it fail? What should the reader do?**

- **Bad:** `Database connection error.`
- **Good:** `Failed to connect to Postgres DB at cluster-abc. Connection timed out after 5000ms. Check network policies or DB CPU load.`
