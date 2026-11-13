# ADR-0003: Guardrails and audit logging as core layers

**Status:** Accepted

## Context
In regulated domains it is not enough for answers to be usually correct. The organization must be able to show that unsafe requests were blocked, that sensitive data was protected, and why a specific answer was given.

## Decision
Make guardrails and audit logging first-class layers with their own interfaces, rather than optional middleware:
- **Input guardrails** run before cache lookup and retrieval.
- **Output guardrails** run before any answer is returned or cached.
- **Audit logging** records every request: caller, timestamp, retrieved sources, model used, guardrail outcomes and the final response status.

## Consequences
- **Better:** consistent enforcement on every path, including cache hits; answers can be explained after the fact.
- **Costs:** added latency for checks; audit data must itself be protected and retained according to policy.

## Alternatives considered
- **Rely on the LLM provider's built-in filters only:** not tailored to domain rules, and gives no audit trail of our own.
