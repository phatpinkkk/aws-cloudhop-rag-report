---
title: "Week 4 Worklog"
date: 2026-06-29
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives

* Study the HotpotQA dataset structure in detail.
* Prepare reusable data structures for retrieval experiments.
* Build initial lexical and dense retrieval baselines.
* Evaluate retrieval quality independently from answer generation.
* Identify limitations of single-pass retrieval on multi-hop questions.

### Tasks to be carried out this week

| Date | Task | Reference Material |
| --- | --- | --- |
| 29/06/2026 | - Inspect the HotpotQA dataset structure.<br>- Analyze question IDs, questions, answers, supporting facts, contexts, and question types.<br>- Identify the information required for retrieval evaluation. | <https://hotpotqa.github.io/> |
| 30/06/2026 | - Design the initial dataset preparation workflow.<br>- Convert HotpotQA contexts into retrievable evidence units.<br>- Align question metadata with candidate evidence and supporting facts.<br>- Save reusable dataset inspection artifacts. | |
| 01/07/2026 | - Implement a lexical retrieval baseline using TF-IDF.<br>- Retrieve evidence at multiple top-k values.<br>- Define retrieval evaluation based on whether annotated supporting evidence is recovered. | |
| 02/07/2026 | - Implement a dense retrieval baseline using Sentence Transformers MiniLM.<br>- Generate vector representations for retrieval candidates.<br>- Compare semantic retrieval behavior with the TF-IDF baseline. | |
| 03/07/2026 | - Evaluate TF-IDF and MiniLM retrieval across different top-k settings.<br>- Inspect retrieval successes and failure cases.<br>- Analyze why single-pass retrieval can miss evidence required for multi-hop questions.<br>- Keep answer generation outside the benchmark to isolate retrieval quality. | |

### Week 4 Achievements

* Completed the initial HotpotQA inspection and preparation workflow.
* Built reusable question, context, and supporting-evidence structures.
* Implemented a TF-IDF lexical retrieval baseline.
* Implemented a MiniLM dense retrieval baseline.
* Evaluated multiple top-k retrieval settings.
* Established a reproducible retrieval-only baseline before adding generation.
* Identified the need for stronger hybrid and multi-hop retrieval methods.