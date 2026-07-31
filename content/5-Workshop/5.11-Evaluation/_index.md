---
title: "Evaluation"
date: 2026-07-31
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

After confirming that the deployment works correctly, CloudHop RAG is evaluated in terms of **retrieval quality, answer quality, and runtime performance**.

All final results use the corrected **HotpotQA Distractor v002** artifacts. The retrieval benchmark contains **500 validation questions**, **4,937 parent documents**, and **8,279 BGE-M3 child vectors**.

Retrieval quality is evaluated at the **supporting-title level**. These measurements should not be confused with the official HotpotQA sentence-level Supporting-Fact EM/F1 metrics. Final answer quality is measured using Exact Match (EM) and token-level F1.

## 1. Retrieval Quality

The retrieval pipeline is evaluated at two stages.

The **Candidate Pool** measures whether the retrieval process discovers the required supporting documents before final evidence selection. **Selected Top-10** evaluates the final set of documents retained for downstream answer generation.

| Metric | Candidate Pool | Selected Top-10 |
| --- | ---: | ---: |
| Mean supporting-title recall | **0.9920** | **0.9740** |
| All supporting titles found | **0.9840** | **0.9480** |
| Recall@5 | 0.5420 | **0.9310** |
| Recall@10 | 0.6270 | **0.9740** |
| Precision@10 | 0.1254 | **0.1948** |
| MRR | 0.7807 | **0.9446** |
| nDCG@10 | 0.5816 | **0.9162** |

The candidate stage achieved very high evidence coverage. All required supporting titles were discovered for **492 of 500 questions**, corresponding to a complete-coverage rate of **0.9840**.

After evidence selection, all supporting titles remained in the final Top-10 for **474 of 500 questions**, while mean supporting-title recall remained high at **0.9740**.

The final selection stage substantially improved the position of relevant evidence. Recall@5 increased from **0.5420 to 0.9310**, MRR from **0.7807 to 0.9446**, and nDCG@10 from **0.5816 to 0.9162**. This indicates that the pipeline successfully concentrates relevant evidence near the top of the final context, although a small amount of supporting evidence is lost when the larger candidate pool is reduced to ten documents.

## 2. End-to-End Answer Quality

A fixed set of **20 questions** was used to evaluate the complete pipeline from retrieval through answer generation. All 20 requests completed successfully.

| Metric | Result |
| --- | ---: |
| Answer EM | **0.7500** |
| Answer F1 | **0.7750** |
| Exact-match answers | **15 / 20** |
| Mean supporting-title recall | **0.9500** |
| All supporting titles found | **0.9000** |
| Recall@5 | **0.9250** |
| Recall@10 | **0.9500** |
| MRR | **0.9667** |
| nDCG@10 | **0.9243** |

The selected evidence contained all required supporting titles for **18 of 20 questions**. Answer EM reached **0.7500**, meaning 15 answers exactly matched the reference answer, while token-level F1 reached **0.7750**.

The difference between evidence coverage and answer accuracy is also important. Strong retrieval increases the likelihood that the necessary information is available, but the final result still depends on how effectively the generation stage interprets and combines that evidence.

## 3. Runtime Performance

Across the 500-question retrieval benchmark, mean pipeline latency was **25.91 seconds per question**, with a median of **25.72 seconds**.

| Stage | Mean latency | Share of total |
| --- | ---: | ---: |
| Query decomposition | 4.32 s | 16.7% |
| Retrieval + adaptive planning | 21.07 s | 81.3% |
| Cross-encoder reranking | 0.53 s | 2.0% |
| Total retrieval pipeline | **25.91 s** | 100% |

The dominant runtime component is **retrieval and adaptive planning**, accounting for approximately 81% of total retrieval latency. Query decomposition contributes about 17%, while cross-encoder reranking accounts for only around 2%.

For the 20-question end-to-end benchmark:

| Component | Mean latency |
| --- | ---: |
| Retrieval pipeline | 26.86 s |
| Final generation | 12.43 s |
| End-to-end | **39.29 s** |

The end-to-end results show that both retrieval and generation contribute meaningfully to response time, with retrieval remaining the larger component.

## 4. Quality and Deployment Trade-off

The results above come from the **quality-oriented evaluation configuration**, which uses wider retrieval, up to three adaptive hops, and cross-encoder reranking.

The AWS deployment uses a lighter configuration designed for the available CPU-based EC2 environment. It reduces BM25 and dense retrieval breadth, limits adaptive retrieval, enables Fast Mode, and disables the cross-encoder reranker.

These configurations should therefore be interpreted as two operating profiles of the same system rather than as a controlled ablation. The evaluation configuration prioritizes retrieval quality, while the deployed configuration places greater emphasis on practical response time and resource usage.

Overall, the evaluation shows that CloudHop RAG can recover most of the supporting evidence required by HotpotQA multi-hop questions and produce strong short-answer results. The main remaining engineering challenge is latency, particularly in the retrieval and adaptive-planning stage.