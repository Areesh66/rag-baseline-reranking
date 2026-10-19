
# RAG Baseline vs. Cross-Encoder Re-Ranking

## 1. Project Overview

This project compares a baseline Retrieval-Augmented Generation (RAG) pipeline with a Cross-Encoder re-ranking pipeline using the **EU AI Act — Regulation (EU) 2024/1689** as the source document.

The objective is to evaluate whether re-ranking retrieved text chunks improves the relevance of the context supplied to a language model and the quality of its answers.

## 2. Document and Setup

- **Source document:** EU AI Act, Regulation (EU) 2024/1689
- **Document size:** 144 pages
- **Number of text chunks:** 315
- **Chunk size:** 350 words
- **Chunk overlap:** 60 words
- **Initial retrieval:** Top 10 chunks
- **Final context:** Top 3 chunks
- **Embedding model:** `sentence-transformers/all-MiniLM-L6-v2`
- **Cross-Encoder:** `cross-encoder/ms-marco-MiniLM-L-6-v2`
- **Answer generation:** Gemini API

The PDF was extracted into page-level text, divided into overlapping chunks, and converted into embeddings for retrieval.

## 3. Baseline RAG Pipeline

The baseline pipeline uses a Bi-Encoder to retrieve the 10 most relevant chunks based on cosine similarity. It selects the first three chunks and passes them to the language model to generate an answer.

**Pipeline:**

`Query → Bi-Encoder Retrieval (Top 10) → Select Top 3 → LLM → Answer 1`

This pipeline is computationally efficient because it does not include an additional re-ranking stage.

## 4. Cross-Encoder Re-Ranking Pipeline

The re-ranking pipeline first retrieves the same 10 candidate chunks using the Bi-Encoder. A Cross-Encoder scores each query–chunk pair, and the candidates are sorted by their scores. The three highest-ranked chunks are passed to the same language model using the same prompt structure to generate Answer 2.

**Pipeline:**

`Query → Bi-Encoder Retrieval (Top 10) → Cross-Encoder Scoring → Re-ranked Top 3 → LLM → Answer 2`

## 5. Architecture Comparison

| Feature | Baseline RAG | Cross-Encoder Re-Ranked RAG |
|---|---|---|
| Initial retrieval | Bi-Encoder | Bi-Encoder |
| Candidate set | Top 10 chunks | Same Top 10 chunks |
| Ranking method | Cosine similarity | Cross-Encoder relevance scores |
| Final context | First 3 retrieved chunks | Re-ranked Top 3 |
| Language model | Same model | Same model |
| Prompt structure | Same prompt template | Same prompt template |
| Main advantage | Lower computational overhead | Can promote relevant chunks |
| Main limitation | Relevant chunks may rank below the Top 3 | Adds latency and cannot recover chunks missing from the initial Top 10 |

## 6. Evaluation Results

### Experiment 1: Prohibited AI Practices

**Query:** What types of artificial intelligence practices are prohibited under the AI Act?

| Chunk ID | Original rank | Re-ranked position |
|---|---:|---:|
| 24 | 8 | 1 |
| 22 | 4 | 2 |
| 284 | 7 | 3 |

The baseline selected chunks **128, 40, and 285**, while the re-ranked pipeline selected chunks **24, 22, and 284**.

The baseline answer did not provide specific prohibited practices because its selected context lacked sufficient detail. Re-ranking promoted more relevant chunks, making the answer more informative. However, the answer remained incomplete and did not provide an exhaustive list.

### Experiment 2: Biometric Dataset-Labelling Exception

**Query:** When is biometric categorisation of biometric datasets allowed, such as labelling or filtering images by hair colour or eye colour?

| Chunk ID | Original rank | Re-ranked position |
|---|---:|---:|
| 11 | 1 | 1 |
| 24 | 8 | 2 |
| 32 | 2 | 3 |

The baseline selected chunks **11, 32, and 31**, while the re-ranked pipeline selected chunks **11, 24, and 32**.

Chunk 24 contains the specific exception concerning lawful labelling, filtering, or categorisation of biometric datasets. The baseline answer missed this exception, while the re-ranked answer identified it and included examples such as sorting images by hair colour or eye colour.

**Finding:** Re-ranking promoted a relevant chunk that was initially outside the Top 3 and improved the answer's coverage of a specific exception.

## 7. Additional Evaluation and Limitations

The workplace emotion-recognition experiment showed that both pipelines identified the prohibition and the exception for medical or safety reasons. The baseline answer was already adequate, so the improvement in the final answer was limited.

The biometric categorisation negation test showed that the baseline answer could be more complete than the re-ranked answer. Therefore, re-ranking does not guarantee better answers for every query.

The answers should be checked against the source document to identify unsupported statements, omissions, and incorrect interpretations.

These experiments use one source document and a limited set of queries. The findings demonstrate examples of retrieval improvement but do not establish that re-ranking always improves factual accuracy or completeness.

## 8. Latency Analysis

The latency of both pipelines was recorded to understand the additional computational cost of Cross-Encoder re-ranking.

| Stage | Baseline (seconds) | Cross-Encoder Re-Ranked (seconds) |
|---|---:|---:|
| Retrieval | 0.0209 | 0.0221 |
| Cross-Encoder scoring and sorting | 0.0000 | 1.0499 |
| LLM generation | 1.3408 | 2.2127 |
| Recorded total | 1.3617 | 3.2847 |

The baseline has lower recorded latency because it selects the Top 3 chunks directly after retrieval. The re-ranked pipeline introduces an additional Cross-Encoder scoring step.

**Timing limitation:** These totals are approximate rather than a controlled end-to-end benchmark because the generation times were obtained from earlier calls. A fair comparison would require repeated measurements under the same conditions.

## 9. Why Is the Cross-Encoder Applied Only to the Top 10?

The document contains 315 chunks. The Bi-Encoder efficiently retrieves the Top 10 candidates, and the Cross-Encoder evaluates only these 10 query–chunk pairs.

Applying the Cross-Encoder to all 315 chunks would require 315 pairwise relevance evaluations per query instead of 10, increasing computational cost and latency.

The two-stage approach balances efficiency and relevance: the Bi-Encoder performs broad retrieval, while the Cross-Encoder refines the candidate ranking. However, re-ranking cannot recover a relevant chunk if the initial Bi-Encoder fails to include it in the Top 10.

## 10. Overall Findings

The experiments indicate that Cross-Encoder re-ranking can improve the relevance of the final context by promoting useful chunks that were initially ranked lower by the Bi-Encoder.

In the prohibited AI practices experiment, chunk 24 moved from rank 8 to rank 1, and the answer became more informative. In the biometric dataset-labelling experiment, chunk 24 moved from rank 8 to rank 2, allowing the answer to identify a specific exception missed by the baseline.

However, better rankings do not guarantee better generated answers. The workplace emotion-recognition experiment showed limited improvement, while the biometric categorisation negation test showed that the re-ranked answer could omit information present in the baseline answer.

Overall, re-ranking can improve retrieved-context relevance, but answer completeness, factual correctness, and latency must also be evaluated.

## 11. Repository Contents

- `RAG_Baseline_Reranking_Cleaned.ipynb` — implementation, experiments, and evaluation.
- `README.md` — project overview, architecture, results, and limitations.

## 12. Reproducibility and API Key

The notebook uses a Gemini API key supplied through Kaggle Secrets. 
