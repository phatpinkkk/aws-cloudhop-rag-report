---
title: "Week 5 Worklog"
date: 2026-07-06
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives

* Improve retrieval quality beyond the initial TF-IDF and MiniLM baselines.
* Introduce stronger lexical and semantic retrieval methods.
* Combine BM25 and BGE-M3 in a hybrid retrieval pipeline.
* Design parent-child document representations for efficient retrieval and useful context.
* Add query decomposition, adaptive multi-hop retrieval, and cross-encoder reranking.

### Tasks to be carried out this week

| Date | Task | Reference Material |
| --- | --- | --- |
| 06/07/2026 | - Review baseline retrieval limitations.<br>- Design an advanced retrieval benchmark with a common retriever interface.<br>- Study BM25, stronger dense retrieval, hybrid retrieval, reranking, and multi-hop retrieval strategies. | |
| 07/07/2026 | - Implement BM25 lexical retrieval.<br>- Introduce **BAAI/bge-m3** as the main dense embedding model.<br>- Compare the complementary roles of keyword matching and semantic similarity. | |
| 08/07/2026 | - Design parent-child document representation.<br>- Divide source documents into smaller child units for dense retrieval.<br>- Preserve parent documents for richer context.<br>- Build child-to-parent mappings. | |
| 09/07/2026 | - Combine BM25 and BGE-M3 retrieval into a hybrid candidate pipeline.<br>- Implement query decomposition for complex questions.<br>- Add adaptive retrieval so additional hops can be triggered when further evidence is needed. | |
| 10/07/2026 | - Add `cross-encoder/ms-marco-MiniLM-L-6-v2` reranking.<br>- Define the complete retrieval flow: decomposition → BM25 + BGE-M3 → adaptive retrieval → candidate aggregation → reranking → selected evidence.<br>- Review retrieved evidence and refine configuration parameters. | |

### Week 5 Achievements

* Implemented BM25 lexical retrieval.
* Adopted BGE-M3 as the main dense embedding model.
* Built a hybrid lexical-semantic retrieval pipeline.
* Implemented parent-child document indexing.
* Added query decomposition and adaptive multi-hop retrieval.
* Added cross-encoder reranking to improve the ordering of retrieved evidence.
* Established the main retrieval architecture used by CloudHop RAG.