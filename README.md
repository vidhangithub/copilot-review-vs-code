# Java PR Review — GitHub Copilot Setup

Pre-PR code review prompts for Java 17 / Spring Boot 4.x microservices.
Drop this folder structure into any repository and your team gets instant,
consistent, 8-pillar code reviews via GitHub Copilot Chat.

---

## Repository Structure

```
your-repo/
└── .github/
    ├── copilot-instructions.md       ← Persistent context for every Copilot session
    └── prompts/
        ├── pr-review.prompt.md       ← Full 8-pillar review
        ├── security-scan.prompt.md   ← Fast security-only scan
        └── resilience-review.prompt.md ← Timeout / circuit breaker audit
```

---

## Setup

### 1. Copy files into your repository
```bash
cp -r .github/ your-repo/.github/
```

### 2. Enable prompt files in VS Code
Ensure you have:
- VS Code 1.99 or later
- GitHub Copilot extension installed and signed in
- Setting enabled: `chat.promptFiles` → `true`

In VS Code settings (`settings.json`):
```json
{
  "chat.promptFiles": true,
  "github.copilot.chat.experimental.prompt-files": true
}
```

### 3. Verify `copilot-instructions.md` is active
Open Copilot Chat and ask: *"What are the coding standards for this project?"*
It should reflect the rules in `copilot-instructions.md`.

---

## Usage

### Full PR Review (run before every PR)
1. Open the file(s) you want reviewed in VS Code
2. Open Copilot Chat (`Ctrl+Shift+I` / `Cmd+Shift+I`)
3. Type:
```
/pr-review
```
Or with explicit file scope:
```
/pr-review #file:PaymentService.java
```

### Security Scan (quick check)
```
/security-scan #file:PaymentController.java
```

### Resilience Review (for service integration code)
```
/resilience-review #file:ExternalPaymentClient.java
```

---

## What the Full Review Covers

| Pillar | Covers | Key "Beyond SOLID" Items |
|--------|--------|--------------------------|
| 1. Design & SOLID | SRP, OCP, LSP, ISP, DIP, DRY, layer violations | Feature envy, Law of Demeter |
| 2. Spring Boot | Bean scoping, transaction proxy, @Async pitfalls | Self-invocation proxy bypass |
| 3. Java 17 | Records, sealed classes, pattern matching, text blocks | BigDecimal for money |
| 4. Resilience | Timeouts, circuit breakers, retry safety, bulkheads | Retry storm prevention |
| 5. Security | Injection, secrets, PII, auth, CORS | PII in logs, sanitised errors |
| 6. Observability | MDC, structured logging, swallowed exceptions, metrics | Async MDC propagation |
| 7. Data Integrity | Transaction isolation, locking, N+1, connection leaks | Race conditions on balances |
| 8. Testing | Behaviour vs implementation, edge cases, integration | Shared state between tests |

---

## Report Output

Every review produces:

- **Executive Summary** with PR verdict (BLOCKED / NEEDS WORK / READY)
- **Severity breakdown** (CRITICAL / MAJOR / MINOR / SUGGESTION counts)
- **Findings table** with location and issue
- **Code fixes** for every CRITICAL and MAJOR finding
- **Positive observations** (reinforces good patterns)
- **Per-pillar pass/fail checklist**
- **Prioritised action list**

---

## Severity Reference

| Severity | Meaning | PR Action |
|----------|---------|-----------|
| CRITICAL | Security hole, data loss, race condition, transaction corruption | Block — must fix now |
| MAJOR | Resilience gap, missing timeout, broken Spring idiom | Fix before merge |
| MINOR | Code smell, missed Java 17 idiom, logging gap | Follow-up ticket |
| SUGGESTION | Modernisation opportunity | Developer discretion |

---

## Team Workflow Recommendation

```
Developer writes code
        ↓
Run /pr-review in Copilot Chat
        ↓
Fix all CRITICAL + MAJOR findings
        ↓
Raise PR
        ↓
Reviewer uses /pr-review on the diff for independent check
        ↓
Merge
```

Making `/pr-review` a **required pre-PR step** in your team working agreement
ensures consistent quality without it depending on any individual reviewer's knowledge.
