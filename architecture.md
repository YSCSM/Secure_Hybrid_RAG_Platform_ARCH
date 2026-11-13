# Architecture

## 1. Context

```mermaid
flowchart LR
    User[Business user / application] -->|question| RAG[Secure Hybrid RAG Platform]
    RAG -->|grounded answer + sources| User
    Docs[(Enterprise documents<br/>policies · procedures · guides)] -->|ingestion| RAG
    RAG <-->|inference| LLM[LLM provider<br/>Azure OpenAI / OpenAI]
    RAG -->|traces| LS[LangSmith]
    RAG -->|audit records| AUD[(Audit store)]
```

The platform sits between users and enterprise knowledge. It never lets the model answer from memory alone: every answer is grounded in retrieved documents and returned with its sources.

## 2. Request flow (online path)

```mermaid
sequenceDiagram
    autonumber
    participant C as Client
    participant A as API
    participant GI as Input guardrails
    participant SC as Semantic cache
    participant R as Hybrid retriever
    participant RR as Reranker
    participant G as Generator (LLM)
    participant GO as Output guardrails
    participant L as Audit log

    C->>A: Question + auth token
    A->>GI: Validate request
    GI-->>A: Allowed / blocked
    A->>SC: Look up similar question
    alt Cache hit
        SC-->>A: Cached answer
    else Cache miss
        A->>R: Retrieve (BM25 + vector)
        R->>RR: Fused candidates
        RR-->>A: Top-k ranked chunks
        A->>G: Grounded prompt + chunks
        G-->>A: Draft answer + citations
        A->>GO: Check grounding and sensitive data
        GO-->>A: Approved answer
        A->>SC: Store answer
    end
    A->>L: Record request, sources, model, guardrail results
    A-->>C: Answer + sources
```

## 3. Ingestion flow (offline path)

```mermaid
flowchart LR
    S[Source documents] --> P[Parse<br/>PDF · Word · text]
    P --> CL[Clean & normalize]
    CL --> CH[Chunk<br/>with metadata]
    CH --> E[Embed]
    E --> Q[(Qdrant<br/>vector index)]
    CH --> B[(BM25<br/>keyword index)]
```

Each chunk keeps metadata (source document, section, version) so that answers can cite sources and so that access or retention rules can be applied at retrieval time.

## 4. Why each component exists

| Component | Without it | With it |
|---|---|---|
| BM25 | Exact identifiers and rare terms are missed | Precise matches on codes, IDs and terminology |
| Vector search | Paraphrased questions are missed | Matches on meaning, not just words |
| Fusion + reranker | Two ranked lists with incompatible scores | One precise list of the most relevant chunks |
| Semantic cache | Every repeated question pays full LLM cost and latency | Repeated questions answered from cache |
| Input guardrails | Prompt injection and out-of-scope requests reach the model | Unsafe requests stopped before inference |
| Output guardrails | Ungrounded or sensitive content can reach users | Answers checked before release |
| Audit log | No way to explain or defend an answer later | Full record per request |
| Tracing | Failures are hard to diagnose | Step-by-step visibility of retrieval and generation |
| Evaluation | Quality regressions go unnoticed | Changes judged by metrics before release |

## 5. Deployment view

```mermaid
flowchart TB
    subgraph App[Application tier]
        API[FastAPI service]
    end
    subgraph Data[Data tier]
        Q[(Qdrant)]
        R[(Redis)]
        AL[(Audit store)]
    end
    subgraph Ext[External services]
        LLM[LLM provider]
        LS[LangSmith]
    end
    API --> Q
    API --> R
    API --> AL
    API --> LLM
    API --> LS
```

The API is stateless, so it scales horizontally. State lives in Qdrant (vectors), Redis (cache) and the audit store.

## 6. Extension points

The same foundation supports further use cases without changing the core layers:

- Codebase question answering
- Root-cause analysis over incident and log data
- Text-to-SQL over governed data sources
- MCP-based tool access for assistants
- Multi-agent workflows built on the retrieval layer
