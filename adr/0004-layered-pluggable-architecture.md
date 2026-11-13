# ADR-0004: Layered architecture with provider-agnostic interfaces

**Status:** Accepted

## Context
The RAG ecosystem changes quickly: vector databases, rerankers, embedding models and LLM providers are all likely to be replaced over the life of a platform. Enterprises also standardize on different providers.

## Decision
Organize the system into layers (api, core, ingestion, retrieval, generation, guardrails, cache, observability, evals). Each layer exposes an interface; concrete providers are injected through configuration.

## Consequences
- **Better:** providers can be swapped (for example Qdrant to another vector store, or OpenAI to Azure OpenAI) without changes to the API or business logic; each layer can be tested in isolation; the platform can be reused as a base for new use cases.
- **Costs:** more upfront structure than a single-script prototype.
