---
title: "🔍 Observability in LLMs"
datePublished: Thu Oct 24 2024 18:30:00 GMT+0000 (Coordinated Universal Time)
cuid: cm2svzwgj000008leg62l8ui6
slug: observability-in-llms
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/Tm3vPSFT3ZU/upload/fd39a90c6eb73c6db1de80bbdc1554b3.jpeg
tags: ai, llm, llama

---

Observability is corner stone for any system. When things break, all the information that you have (be it log, traces, metric etc.) aid you to find root causes quickly and help remediate the issue.

LLMs break; due to hallucinations, not having enough information and hence providing irrelevant data, infinite chain of thought loops etc.

Hence, there is a need for observing how the LLM models behave. This is imperative for enterprises having custom LLMs deployed as RAG, Fine tune LLMs etc. along with the observability we have in the traditional software like Splunk, ELK, Datadog etc.

📚 Observability metrics

1️⃣ Traces

Similar to microservice tracing, traces in LLM track the flow of query that was utilized to give the response using trace/span IDs. If it was a RAG application, it would go into the information retrieval, Vector DB query, document used for giving the response.

2️⃣ Feedback

If you have noticed in ChatGPT, there is a Like/Dislike button. This is typically used to help understand how latest models are behaving with respect to user feedback. This information aids the LLM providers to fine-tune or backtrack for any LLM regression.

3️⃣ Eval

Think of this as a unit test case for LLMs. You evaluate the performance of the system by providing expected answers that the LLM should provide. If the actual output from the answer is similar to expected result, then the LLM system is good enough for deployment. If not, then it needs to be looked into and fine-tuned.

🛠️ LLM observability tools:

1️⃣ Langfuse

✨ Features:

\- Traces and spans for each step in your LLM pipeline

\- Managing prompt versions. Can be used as seperate envs like dev, prod etc

\- Evals: Scoring your LLM &/ RAGs for correctness

🔗 Link: [https://langfuse.com/docs](https://langfuse.com/docs)

2️⃣ Traceloop

✨ Features:

1\. Use OpenTelemetry that helps you use existing o11y infra like Grafana, Dynatrace etc.

2\. Work with user feedback for LLM evals

3\. Support for Modals, Frameworks (Langchain etc) and Vector DBs

🔗 Link: [https://www.traceloop.com/docs/introduction](https://www.traceloop.com/docs/introduction)

3️⃣ Phoenix Arize

✨ Features:

1\. OpenTelemetry support

2\. Support for frameworks and SDKs

3\. Evals: Evals are hard for LLM. Phoenix solves this by providing Evals template and running it at span/trace level.

🔗 Link: [https://docs.arize.com/phoenix](https://docs.arize.com/phoenix)

4️⃣ Arize (Enterprise equivalent to Pheonix Arize)

✨ Features (on top of Phoenix arize):

\- Experiments and dataset: Change the input prompt as well as dataset to check LLMs regressions or issues.

\- Arize Copilot: Ask it questions regarding your LLM events

🔗 Link: [https://docs.arize.com/arize](https://docs.arize.com/arize)