---
draft: true
date: 2024-01-31
categories:
  - Performance Engineering
  - Building with LLMs
---

# Performance Engineering with LLM Applications - Part 2: Strategies for Improving Performance

## Background

Performance is critical for any application, but especially for those built on top of Large Language Models (LLMs). Language models introduce bottlenecks in the request-response cycle that are not present in traditional web applications, such as model inference speed. These bottlenecks can be exacerbated by the fact that most applications use third-party model vendors, which means that the application developer has limited control over the model itself. Performance issues can be difficult to debug and fix. These issues compound over time and lead to a poor user experience.

At Clarative, we’re building a self-documenting AI-native tool for enterprise data management and analysis. We’ve spent a lot of time thinking about how to build a performant, model-agnostic application. In this post, we’ll share some of our learnings from debugging and improving the performance of our application. This post is meant to be a helpful guide for anyone building an LLM application.

### Contents

This post is split into the following sections: [TODO]: Add links to sections

## Parallelizing I/O operations

- Use the dependency graph to understand which I/O operations can be parallelized
- `asyncio.gather` and `asyncio.as_completed`

## Batching requests

- Certain endpoints, such as OpenAI embeddings, support batching
- Fuzzy batching via asking the model to predict multiple outputs at once

## Streaming model responses

- Tradeoff between streaming and validation

## Caching model responses

## Selecting the model based on the task

## Fine-tuning a smaller model on a specific task

## Conclusion

### References
