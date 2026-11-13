# ADR-0001: Hybrid retrieval (BM25 + vector) instead of vector-only search

**Status:** Accepted

## Context
Enterprise documents in healthcare and insurance are full of exact identifiers: policy numbers, plan codes, procedure codes, form names. Vector embeddings capture meaning well but often rank exact-term matches poorly. Keyword search (BM25) does the opposite: precise on exact terms, weak on paraphrased questions.

## Decision
Run BM25 and vector search (Qdrant) in parallel, fuse the two result lists into one candidate set, then apply a reranker to select the final top-k chunks passed to the model.

## Consequences
- **Better:** higher recall across both paraphrased and exact-term questions; reranking improves precision of what reaches the model.
- **Costs:** two indexes to maintain and keep in sync; reranking adds latency per request (mitigated by semantic caching, ADR-0002).
- **Measured by:** retrieval metrics in the evaluation suite (ADR-0005).

## Alternatives considered
- **Vector-only:** simplest, but misses exact identifiers.
- **BM25-only:** misses paraphrased questions.
- **Larger embedding model alone:** improves semantic quality but does not reliably solve exact-term matching.
