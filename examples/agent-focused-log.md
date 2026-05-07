# Dev Log - 2026-05-07

## 2026-05-07 14:10 CST - login-rate-limit-redis

Agent: Codex
Status: done
Task: implement login failure rate limiting
Audience: agent-handoff

### Summary
- Replaced in-memory login failure counter with Redis-backed counter because the service runs multiple instances.

### Decisions
- Use Redis instead of process memory.
- Redis key pattern: `login:fail:{user_id}`.
- TTL: 15 minutes.
- Clear failure key after successful login.

### Files
- `src/auth/login.ts`: added Redis counter update and reset logic.
- `src/auth/rateLimit.ts`: added login failure limiter.
- `tests/auth/login.test.ts`: added failure count, TTL, and reset coverage.

### Verification
- Command: `npm test -- tests/auth/login.test.ts`
- Result: passed

### Continuation
- Safe to continue from current implementation.
- If adding IP-based limits, avoid mixing user and IP counters in the same key namespace.
