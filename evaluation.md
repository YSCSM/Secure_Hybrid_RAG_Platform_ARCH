# Evaluation Approach

## What is measured

| Stage | Question it answers | Example metrics |
|---|---|---|
| Retrieval | Did we find the right content? | Context precision, context recall, hit rate @k |
| Generation | Is the answer supported by that content? | Faithfulness / groundedness |
| End to end | Does the answer address the question? | Answer relevance, correctness against reference |
| Operations | Is it fast and affordable enough? | Latency, cache hit rate, tokens per request |

## How it runs
1. A curated dataset of representative questions, with expected sources and reference answers.
2. The evaluation suite runs the full pipeline over the dataset.
3. Results are compared with the previous baseline.
4. A drop beyond tolerance blocks the change (ADR-0005).

## Experiments this enables
- Vector-only vs hybrid retrieval
- With vs without reranking
- Chunk size and overlap
- Embedding and LLM model choices
- Cache similarity thresholds

## Results
| Configuration | Context precision | Faithfulness | Answer relevance | p95 latency |
|---|---|---|---|---|
| Vector-only | TBD | TBD | TBD | TBD |
| Hybrid | TBD | TBD | TBD | TBD |
| Hybrid + reranker | TBD | TBD | TBD | TBD |

> Fill this table from real evaluation runs before publishing, or remove it. Do not estimate.
