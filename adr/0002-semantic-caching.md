# ADR-0002: Semantic cache in front of retrieval and generation

**Status:** Accepted

## Context
In enterprise settings many users ask the same questions in different words. Without caching, each one pays the full cost of retrieval, reranking and an LLM call.

## Decision
Use Redis as a semantic cache. Incoming questions are embedded and compared with cached questions; above a similarity threshold, the cached answer is returned. Cached answers are written only after passing output guardrails.

## Consequences
- **Better:** lower latency and LLM cost for repeated questions.
- **Risks:** a threshold set too low returns answers to questions that only look similar; stale answers after documents change.
- **Mitigations:** conservative similarity threshold, cache entries expire (TTL) and are invalidated when the source index is updated; cache keys can be scoped by user or tenant where access rules differ.

## Alternatives considered
- **Exact-match cache:** safe but rarely hits, because wording varies.
- **No cache:** simplest, highest cost and latency.
