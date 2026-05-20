# GitHub Copilot Instructions

## Project Context
- **Language**: Java 17
- **Framework**: Spring Boot 4.x
- **Domain**: Financial services microservices
- **Architecture**: Microservices on AKS with Istio service mesh

## Code Standards — Always Apply

### Non-Negotiable Rules
- Use `BigDecimal` (never `double`/`float`) for all monetary values
- All monetary `BigDecimal` values must use `setScale(2, RoundingMode.HALF_EVEN)`
- All external HTTP/DB calls must have explicit timeouts configured
- No secrets, credentials, or PII in log statements at any level
- Constructor injection only — no `@Autowired` field injection
- `@Transactional` on service layer only — never on controllers or repositories
- All controller request bodies must have `@Valid` annotation

### Java 17 Preferences
- Prefer `record` for immutable data carriers (DTOs, response objects)
- Use `sealed` classes for closed type hierarchies (result types, event types)
- Use pattern matching `instanceof Type t` instead of cast-after-instanceof
- Use switch expressions over switch statements
- Use text blocks for multi-line SQL, JSON, XML

### Spring Boot Preferences
- `@ConfigurationProperties` over scattered `@Value` annotations
- `readOnly = true` on all read-only `@Transactional` methods
- Resilience4j `@CircuitBreaker` + `@Retry` on all external service calls
- MDC correlation ID (`traceId`, `correlationId`) on all inbound requests

### Testing Preferences
- Assert behaviour and outcomes, not mock interactions
- Use `@DataJpaTest` + Testcontainers for persistence tests
- Test names: `should_[outcome]_when_[condition]()`

## Available Prompt Files

Run these in Copilot Chat before raising a PR:

| Command | Purpose |
|---------|---------|
| `/pr-review` | Full 8-pillar review — run before every PR |
| `/security-scan` | Fast security-only scan |
| `/resilience-review` | Timeout, circuit breaker, retry audit |
