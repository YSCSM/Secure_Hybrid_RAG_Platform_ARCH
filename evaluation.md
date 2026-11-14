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

From the last full evaluation run (31 October 2026), comparing three retrieval configurations on the same data.

- **Data:** a knowledge base for a fictional company (12 documents, 16 chunks) and 32 hand-written questions: 25 answerable and 7 deliberately unanswerable. The company is made up so the model cannot answer from pre-training, which means the scores reflect retrieval and grounding rather than model memory.
- **Models:** `gpt-4o-mini` writes the answers and also judges them (RAGAS). Retrieval passes the top 5 chunks to the model.
- **Cost:** about $0.17 for the full run, mostly judge calls.

| Configuration | Doc recall @5 | MRR | Context precision | Faithfulness | Answer relevance | Unanswerable refused at retrieval | Mean latency |
|---|---|---|---|---|---|---|---|
| Vector-only | 100% | 0.92 | 0.89 | 0.97 | 0.83 | 29% | 1.21 s |
| Hybrid (BM25 + vector, RRF) | 100% | 0.93 | 0.90 | 1.00 | 0.82 | 29% | 1.04 s |
| Hybrid + reranker | 100% | 0.98 | 0.99 | 0.98 | 0.84 | 57% | 1.26 s |

Doc recall, MRR and refusal at retrieval are deterministic metrics that need no LLM. Context precision, faithfulness and answer relevance are RAGAS scores (0 to 1) over the 25 answerable questions. In all three configurations the model declined every unanswerable question that got past retrieval, so end-to-end refusal was 100%.

**What the run showed**
- **Reranking earns its place.** Compared with vector-only, it raised context precision from 0.89 to 0.99 and MRR from 0.92 to 0.98, and it doubled the unanswerable questions refused before any LLM call (29% to 57%). It added about 0.2 s over hybrid without reranking. It did not improve recall, because vector search already found every reference document in a corpus this small.
- **Hybrid without reranking scored about the same as vector-only.** On this question set, vector search already found the documents for the exact-code questions that BM25 is meant to help with.
- **The evaluation changed a setting.** A threshold sweep showed the original reranker cut-off (0.1) was discarding three correct documents. Lowering it to 0.01 raised doc recall from 92% to 100% without changing refusals.
- **Relevance is not answerability.** Unanswerable questions about the company ("Who is the CEO?") look relevant to retrieval, so no threshold can reject them without losing real answers. The grounded prompt catches them: 100% refused end to end, against 57% at retrieval.
- **Weak spots:** two answers were judged partly unsupported (faithfulness 0.67 and 0.75), and one two-part question was only half answered because the retrieved context covered one part.

**Limits**
- 32 questions is a small sample: one answerable question moves recall by 4 points, and repeated runs moved some RAGAS scores by up to 0.04. Treat smaller differences as noise.
- The judge is from the same model family as the generator and may be lenient toward it.
- Recall is counted per document, not per chunk.
- Latency is the mean per question; the sample is too small for a meaningful p95.
- The semantic cache and guardrails are not yet part of the evaluated pipeline, so there are no results for cache hit rate or guardrail behaviour.
