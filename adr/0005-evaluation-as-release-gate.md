# ADR-0005: Automated RAG evaluation as a release gate

**Status:** Accepted

## Context
Changes to chunking, prompts, retrievers or models can silently reduce answer quality. Manual spot checks do not scale and are not repeatable.

## Decision
Maintain an evaluation dataset of representative questions with expected sources and reference answers. Run automated RAG evaluation on every significant change and compare against the previous baseline. A drop beyond an agreed tolerance blocks the release.

## Consequences
- **Better:** quality changes are measured, not guessed; tradeoffs (for example, faster but less accurate) become explicit decisions.
- **Costs:** the evaluation dataset must be curated and kept current; evaluation runs add time and LLM cost to the release process.
