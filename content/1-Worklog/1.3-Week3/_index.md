---
title: "Week 3 Worklog"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives

* Understand the basic machine learning lifecycle and how AI applications can be deployed in cloud environments.
* Learn text embeddings, vector similarity, and semantic retrieval.
* Understand Retrieval-Augmented Generation and its role in grounding LLM answers.
* Study the difference between single-hop and multi-hop question answering.
* Define the CloudHop RAG project and select a suitable benchmark dataset.

### Tasks to be carried out this week

| Date | Task | Reference Material |
| --- | --- | --- |
| 22/06/2026 | - Review the machine learning lifecycle:<br>&emsp; + Data preparation<br>&emsp; + Model inference<br>&emsp; + Evaluation<br>&emsp; + Deployment<br>- Explore how AI/ML workloads can be integrated with AWS infrastructure. | <https://cloudjourney.awsstudygroup.com/> |
| 23/06/2026 | - Learn text embeddings and vector representations.<br>- Study cosine similarity and semantic search.<br>- Compare lexical keyword matching with semantic retrieval. | |
| 24/06/2026 | - Learn Retrieval-Augmented Generation.<br>- Study the main RAG flow:<br>&emsp; + Retrieve external evidence<br>&emsp; + Build context<br>&emsp; + Generate an answer<br>- Understand how retrieved evidence can improve answer grounding. | |
| 25/06/2026 | - Study single-hop and multi-hop question answering.<br>- Examine why some questions require evidence from multiple documents.<br>- Define the initial CloudHop RAG flow: question → retrieval → evidence selection → generation → answer. | |
| 26/06/2026 | - Study the HotpotQA benchmark.<br>- Examine bridge and comparison questions, candidate contexts, answers, and supporting facts.<br>- Select HotpotQA Distractor as the main controlled benchmark for the project. | <https://hotpotqa.github.io/> |

### Week 3 Achievements

* Understood the main stages of the machine learning lifecycle.
* Learned how text embeddings represent semantic information.
* Understood the difference between lexical and semantic retrieval.
* Learned the principles of Retrieval-Augmented Generation.
* Understood why multi-hop questions require complementary evidence from multiple documents.
* Defined the CloudHop RAG project as a multi-hop RAG system.
* Selected HotpotQA as the main benchmark for retrieval and answer evaluation.