# Security & Compliance Model

Designed with regulated data (for example PHI and PII) in mind.

## Principles
1. **Least privilege:** callers are authenticated at the API; retrieval can be filtered by document metadata so users only receive content they are entitled to.
2. **Defense in depth:** guardrails before the model, after the model, and before anything is cached.
3. **Traceability:** every answer can be traced to its sources, model and guardrail decisions.
4. **No secrets in code:** credentials and keys come from environment configuration or a secrets manager.

## Controls

| Risk | Control |
|---|---|
| Prompt injection | Input guardrails; system prompt separated from retrieved content |
| Sensitive data leakage | Output guardrails for sensitive patterns; masking at ingestion where required |
| Hallucinated answers | Grounded prompting with citations; grounding checks on output; evaluation suite |
| Unauthorized access | Authentication at the API; metadata-based retrieval filtering |
| Cache leakage across users | Cache scoped by tenant or access context where rules differ |
| Lack of accountability | Audit log per request; LangSmith tracing |
| Vulnerable dependencies | Dependency and code scanning in CI (for example Snyk) |

## Audit record (example fields)
`request_id`, `timestamp`, `caller_id`, `question_hash`, `retrieved_source_ids`, `model`, `guardrail_results`, `cache_hit`, `latency_ms`, `response_status`
