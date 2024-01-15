---
draft: true
date: 2024-01-31
categories:
  - Performance Engineering
  - Building with LLMs
---

# Performance Engineering with LLM Applications - Part 1: Identifying Bottlenecks

## Background

Performance is an important consideration for any application, especially for those built on top of Large Language Models (LLMs). Performance issues are difficult to debug and fix, leading to problems that compound over time and lead to a poor user experience. Further, language models introduce bottlenecks in the request-response cycle that are not present in traditional web applications, such as model inference speed. These bottlenecks can be exacerbated by the fact that most applications use third-party model vendors, which means that the application developer has limited control over the model itself.

At Clarative, we’re building a self-documenting, AI-native tool for enterprise data management and analysis. We’ve spent a lot of time thinking about how to build a performant, model-agnostic application. In this post, I’ll share some of our learnings from debugging and improving the performance of our application. This post is meant to be a helpful guide for anyone building a Python LLM application. Before diving in, a quick primer:

**What is profiling?**

Profiling is a technique that can be used to identify performance bottlenecks in an application. More specifically, it’s a form of dynamic analysis that measures the performance of an application as it runs.

**Why Python `asyncio`?**

Python is a popular choice for building LLM applications because of its ease of use and large ecosystem of libraries, particularly in the data science, machine learning, and AI/NLP spaces. Python is . As a result, there are a number of tools and techniques for improving performance. One of the most popular is the `asyncio` module, which is a library for writing asynchronous code in Python.

Whether you’re using a third-party model vendor or building with your own model, calls to the language model will need to run asynchronously because of how computationally expensive model inference is.

As a result, applications built on top of LLMs are typically I/O bound, meaning that they spend most of their time waiting for I/O operations (e.g. network requests, file reads, etc.) to complete. In order to improve performance, these applications typically leverage asynchronous code so that they can perform other tasks while waiting for expensive I/O operations to complete.

## Limitions of the builtin `cProfile` module

`cProfile` is a builtin Python module that can be used to profile Python code. It’s generally a good choice because it’s included in the standard library and is relatively easy to use. Unfortunately, cProfile doesn’t support profiling asynchronous code. As a result, we’ll need to explore some other options.

## Getting a bird’s eye view with `pyinstrument`

**`pyinstrument`** supports profiling asynchronous code and is relatively easy to use.

According to the [documentation](https://pyinstrument.readthedocs.io/en/latest/how-it-works.html#async-profiling):

> This async support works by tracking the ‘context’ of execution, as provided by the built-in contextvars module. When you start a Profiler with the async_mode enabled or strict (not disabled), that Profiler is attached to the current async context.

## Drilling down further with `yappi`

**`yappi`** is another Python profiler that supports profiling asynchronous code.

**Decorator for profiling an async function**

```python
from asyncio import iscoroutinefunction
from functools import wraps
from typing import Callable, Optional
import yappi

def profile():
    """Decorator to profile a function.

    Example:
    @profile()
    async def my_async_function():
        ...
    """
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            yappi.set_clock_type("wall")  # Use set_clock_type("cpu") for cpu time
            yappi.start(builtins=True)
            if iscoroutinefunction(func):
                output = await func(*args, **kwargs)
            else:
                output = func(*args, **kwargs)
            yappi.stop()
            stats = yappi.get_func_stats()
            stats.print_all()

        return wrapper
    return decorator
```

## Visualizing profile logs in **`kcachegrind`**

### Understanding the dependency graph

## Conclusion

### General strategies for improving performance

1. **Choose an end-to-end workflow to optimize.** This will help you focus your efforts and avoid premature optimization.
1. **Get a baseline for the performance of your workflow.** Having a baseline will help you determine if your changes are actually improving performance.
1. **Profile an end-to-end workflow using realistic inputs.** This should mirror a real-world scenario, e.g. a user’s experience.
1. **Focus on the code that you control.** For example, if you’re using a third-party model vendor, you may not be able to optimize the model inference speed.
1. **Identify the largest bottlenecks and low-hanging fruit.** This will help you maximize the impact of your efforts.

### Next steps

The next post will share some strategies for applying the learnings from profiling to improve performance.

### References

- https://www.roguelynn.com/words/asyncio-profiling/
- https://pyinstrument.readthedocs.io/en/latest/how-it-works.html#async-profiling
