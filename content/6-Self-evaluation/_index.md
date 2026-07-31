---
title: "Self-evaluation"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

During my internship with **AWS First Cloud AI Journey**, I had the opportunity to work on a complete AI project rather than only individual experiments. My main contribution to **AWS CloudHop RAG** was on the retrieval and evaluation side, including benchmark preparation, lexical and dense retrieval, multi-hop retrieval, debugging, and performance analysis. Through this work, I became more confident in approaching unfamiliar technical problems, validating experimental results, and understanding how an AI pipeline fits into a larger cloud application.

I think I performed well overall, especially in the technical areas I was responsible for. My strongest areas were learning new concepts quickly, problem solving, and working carefully with evaluation results. At the same time, I do not consider my performance perfect. Some experiments took longer than necessary, and I could have communicated intermediate problems to the team earlier instead of investigating them independently for too long. I also had less hands-on involvement in some parts of the AWS deployment than I would have liked.

### Self-evaluation

| No. | Criteria | Self-evaluation | Good | Fair | Average |
| --- | --- | --- | :---: | :---: | :---: |
| 1 | **Professional knowledge and technical skills** | Developed a stronger understanding of RAG, lexical and semantic retrieval, embeddings, multi-hop retrieval, evaluation, and how these components interact within an AWS application. | ✅ | ☐ | ☐ |
| 2 | **Learning ability** | Learned new retrieval methods, evaluation techniques, and AWS concepts quickly and applied them directly to the project. | ✅ | ☐ | ☐ |
| 3 | **Proactiveness** | Explored retrieval and evaluation approaches beyond the initial baseline and investigated unexpected results independently. | ✅ | ☐ | ☐ |
| 4 | **Discipline and time management** | Completed the main assigned work, but some experiments and benchmark runs could have been planned and prioritized more efficiently. | ☐ | ✅ | ☐ |
| 5 | **Communication** | Shared results and technical findings with teammates, but I could improve by communicating blockers and intermediate progress earlier. | ☐ | ✅ | ☐ |
| 6 | **Teamwork** | Worked effectively with teammates across retrieval, backend, frontend, and AWS deployment tasks, and contributed to validating the integrated system. | ✅ | ☐ | ☐ |
| 7 | **Problem solving** | Investigated retrieval failures, data issues, evaluation behavior, and performance bottlenecks by tracing problems through the pipeline instead of relying only on final metrics. | ✅ | ☐ | ☐ |
| 8 | **Contribution to the project** | Contributed mainly to benchmark preparation, retrieval development, evaluation, debugging, result analysis, and final system validation. | ✅ | ☐ | ☐ |
| 9 | **Overall performance** | Performed well in my main responsibilities and contributed reliably to the project, while identifying deployment experience, communication, and experimentation efficiency as areas to improve. | ✅ | ☐ | ☐ |

### Personal Contribution

My main responsibility in CloudHop RAG was **retrieval and evaluation**. I worked on preparing and validating the HotpotQA benchmark, developing and comparing retrieval approaches, improving the multi-hop retrieval pipeline, and analyzing both retrieval quality and runtime behavior.

I also contributed to the final validation of the system by checking whether the retrieved evidence and generated answers remained reasonable after the pipeline was integrated with the AWS application. This required understanding how my work interacted with other components even though I was not the main person responsible for every part of the deployment.

Working on evaluation also made me more careful about how I interpret experimental results. I learned to look beyond a single metric and inspect the data, retrieved candidates, selected evidence, and final answers when something behaved unexpectedly.

### Difficulty and How It Was Addressed

One of the main difficulties I faced was balancing **retrieval quality with the practical limits of the deployment environment**.

The strongest evaluation configuration could use larger candidate sets, additional retrieval hops, reranking, and more model calls. These choices helped retrieval quality, but they also increased latency and computational requirements. The AWS resources available to our team could not support every quality-oriented component at the same level used during experimentation.

Instead of forcing one configuration to serve both purposes, we separated the two needs. The stronger configuration was kept for controlled evaluation, while the deployed application used a lighter configuration with fewer candidates, fewer retrieval steps, and less expensive processing.

This was an important lesson for me because it showed that there is rarely one configuration that is simply "best." In an experiment, the priority may be retrieval quality, while a deployed application also has to consider latency, cost, available infrastructure, and reliability. Finding a reasonable balance between these factors is part of the engineering work.

### Areas for Improvement

The first area I want to improve is **AWS deployment and cloud infrastructure experience**. By the end of the internship, I understood the overall architecture much better, but my main work was still concentrated on retrieval and evaluation. In future projects, I would like to take more direct responsibility for backend deployment, networking, permissions, API integration, and production troubleshooting.

I also want to improve how I **manage experimentation time**. During retrieval development, it was easy to explore many methods and configurations. Some experiments were useful, while others had limited impact on the final system. In future work, I want to define clearer hypotheses before running experiments and prioritize the ones most likely to answer an important question.

Another area is **communication during technical problem solving**. I sometimes prefer to investigate an issue deeply before discussing it with others. While this helps me work independently, sharing blockers and intermediate findings earlier could make collaboration more efficient when the problem affects other parts of the system.

### Teamwork

Working with my team went well because we gradually divided responsibilities according to our strengths. I focused more heavily on retrieval, benchmark preparation, and evaluation, while other members contributed more directly to different parts of the backend, frontend, and AWS deployment.

Even with this division, our work was closely connected. I regularly shared retrieval and evaluation findings with the team and helped validate whether the final application was returning the expected evidence and answers.

Looking back, I could have spent more time working directly on components outside my main responsibility. The division of work helped us move faster, but I also learned that specialization should not prevent each member from understanding the broader system.

### Reflection

The biggest lesson I took from this internship is that **being good at the AI part is only one part of being an AI engineer**.

Retrieval and evaluation were the areas where I felt most comfortable, but CloudHop RAG also depended on data preparation, backend development, APIs, AWS infrastructure, permissions, deployment, latency, and coordination between different components. An AI method that performs well in an experiment is not automatically a good deployed solution.

The internship helped me become more confident in technical problem solving while also showing me where I still need to grow. Going forward, I want to keep strengthening my machine learning and evaluation skills while becoming more capable of taking an AI system from experimentation to a complete and reliable application.