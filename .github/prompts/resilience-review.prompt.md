---
mode: 'agent'
description: 'Resilience and fault tolerance review for Java Spring Boot microservices. Use when reviewing service-to-service calls, external integrations, or any code that calls downstream systems.'
---

# Resilience & Fault Tolerance Review

You are reviewing a Spring Boot microservice for resilience gaps.
Focus exclusively on fault tolerance, timeouts, and failure handling.
This is especially critical for financial services where downstream failures must not cascade.

## Review Checklist

### Timeouts — CRITICAL if missing on any external call

For every external call (HTTP, gRPC, database, message queue, cache):
- [ ] Is a **connect timeout** explicitly configured?
- [ ] Is a **read/response timeout** explicitly configured?
- [ ] Are JVM/library defaults NOT relied upon? (Default is often infinite)

Flag the specific client and method. Example:
```java
// CRITICAL — no timeout
RestTemplate restTemplate = new RestTemplate();

// Correct
HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
factory.setConnectTimeout(3000);
factory.setReadTimeout(5000);
RestTemplate restTemplate = new RestTemplate(factory);
```

### Circuit Breakers — MAJOR if missing on external service calls
- [ ] Is `@CircuitBreaker(name = "...", fallbackMethod = "...")` present?
- [ ] Is the fallback method meaningful — not an empty return or silent null?
- [ ] Are CB thresholds configured in `application.yml` (not relying on defaults)?

### Retry Safety — CRITICAL if retrying non-idempotent operations
- [ ] Is `@Retry` applied only to idempotent operations (GET, read, lookup)?
- [ ] For non-idempotent (payment submit, trade execution): is deduplication / idempotency key in place?
- [ ] Is `maxAttempts` bounded?
- [ ] Is exponential backoff + jitter configured?
```yaml
resilience4j.retry:
  instances:
    paymentLookup:
      maxAttempts: 3
      waitDuration: 500ms
      enableExponentialBackoff: true
      exponentialBackoffMultiplier: 2
      retryExceptions:
        - org.springframework.web.client.ResourceAccessException
```

### Bulkheads — MAJOR if slow/high-volume calls share default pool
- [ ] Are slow downstream calls isolated in a dedicated thread pool?
- [ ] Is `@Bulkhead` annotation present with named config?

### Exception Handling — MAJOR if swallowed
- [ ] Are ALL catch blocks logging the exception with context before rethrowing?
- [ ] Is there a meaningful fallback, not a silent empty result?
- [ ] Are `CompletableFuture` chains handling `exceptionally()` or `handle()`?

---

## Report Format

```
RESILIENCE REVIEW — [Service / ClassName]

EXTERNAL CALLS IDENTIFIED:
1. [Class:method] → [downstream target]
   Timeout: ✅/❌ | Circuit Breaker: ✅/❌ | Retry Safe: ✅/❌

FINDINGS:
| # | Severity | Location | Issue | Fix |
|---|----------|----------|-------|-----|

VERDICT: [ ❌ BLOCKED | ⚠️ NEEDS WORK | ✅ READY ]
```
