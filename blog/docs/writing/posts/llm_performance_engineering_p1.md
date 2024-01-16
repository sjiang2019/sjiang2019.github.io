---
draft: true
date: 2024-01-31
categories:
  - Performance Engineering
  - Building with LLMs
---

# Building with LLMs: Profiling Python `asyncio`

## Background

Performance is an incredibly important consideration for any application built on top of Large Language Models (LLMs). In their current form, LLMs are inherently slow, which impacts user experience. Moreover, since most applications use third-party model vendors, developers often face constraints, like rate limits and limited customization capabilities, that make performance optimization a challenging task.

This post will delve into strategies for debugging and enhancing performance in Python asyncio applications.

**What is profiling?**

Profiling is a technique that can be used to identify performance bottlenecks in an application. More specifically, it’s a form of dynamic analysis that measures the performance of an application as it runs.

**Why Python `asyncio`?**

Python is a popular choice for building LLM applications because of its ease of use and large ecosystem of libraries, particularly in the data science, machine learning, and AI/NLP spaces.

The asyncio module is used to write asynchronous code in Python, enabling efficient management of I/O-bound operations. For instance, it allows for non-blocking network calls to a remotely hosted language model.

**The I/O bound nature of LLM applications**

LLM applications typically involve communicating with a language model hosted on a remote server, either by a model provider or within the organization. This setup means that the application's performance is largely dependent on the efficiency of I/O operations, such as network requests. Asynchronous programming, facilitated by asyncio, is key to managing these operations, thereby enhancing the overall performance and responsiveness of the application.

## Example

Consider the following example of an application that fetches data for a user, preprocesses the data, makes parallel API calls, processes the request with a language model, postprocesses the output, and stores the results.

```python
import asyncio


async def unnecesary_io_operation():
    # Simulating an unnecessary I/O operation
    await asyncio.sleep(1.0)
    print("Unnecessary I/O operation")
    return True


async def fetch_user_data(user_id):
    # Simulating a database/API call with sleep
    await asyncio.sleep(2.0)
    print(f"Fetched data for user {user_id}")
    return {'user_id': user_id, 'data': 'Sample data'}


async def preprocess_input(data):
    # Simulating input preprocessing with an inefficient loop
    await asyncio.sleep(1.0)
    print("Preprocessed input data")
    return data


async def make_parallel_api_calls(data):
    # Simulating parallel API calls
    await asyncio.gather(
        asyncio.sleep(1.0),
        asyncio.sleep(2.0),
    )
    print("Made parallel API calls")
    return data


async def process_request_with_llm(data):
    # Simulating LLM processing
    await asyncio.sleep(3.0)
    print("Processed request with LLM")
    return {'processed_data': data['data'] + ' + LLM processed'}


async def postprocess_output(data):
    # Simulating output postprocessing with resource-intensive operation
    await asyncio.sleep(1.0)
    print("Postprocessed output data")
    await unnecesary_io_operation()
    return {'final_output': processed}


async def store_results(processed_data):
    # Simulating a database write or external API call with sleep
    await asyncio.sleep(2.0)
    print("Stored results")
    return True


async def handle_user_request(user_id):
    user_data = await fetch_user_data(user_id)
    preprocessed_data = await preprocess_input(user_data)
    api_data = await make_parallel_api_calls(preprocessed_data)
    processed_data = await process_request_with_llm(api_data)
    postprocessed_data = await postprocess_output(processed_data)
    result = await store_results(postprocessed_data)
    return result


async def main():
    await handle_user_request('user123')
```

## Limitions of the builtin `cProfile` module

`cProfile` is a builtin Python module that can be used to profile Python code. It’s generally a good choice because it’s included in the standard library and is relatively easy to use. Unfortunately, cProfile doesn’t support profiling asynchronous code. As a result, we’ll need to explore some other options.

## Getting a bird’s eye view with `pyinstrument`

**`pyinstrument`** is a powerful profiling tool for Python applications, including those using asyncio. It can provide an easy-to-understand overview of where an application spends most of its time.

### How to use `pyinstrument`

```python
from pyinstrument import Profiler

async def main():
    # your asyncio code here

profiler = Profiler()
profiler.start()

# run your asyncio main function
await main()

profiler.stop()
print(profiler.output_text(unicode=True, color=True))

```

### Running `pyinstrument` on our example

```plaintext
Fetched data for user user123
Preprocessed input data
Made parallel API calls
Processed request with LLM
Postprocessed output data
Unnecessary I/O operation
Stored results

  _     ._   __/__   _ _  _  _ _/_   Recorded: 21:58:18  Samples:  9
 /_//_/// /_\ / //_// / //_'/ //     Duration: 12.009    CPU time: 0.010
/   _/                      v4.6.1


12.009 _UnixSelectorEventLoop._run_once  asyncio/base_events.py:1845
|- 11.009 Handle._run  asyncio/events.py:78
|     [5 frames hidden]  asyncio, IPython
|        10.007 ZMQInteractiveShell.run_ast_nodes  IPython/core/interactiveshell.py:3367
|        `- 10.007 <module>  ../../../../../var/folders/_n/2hcmx0kn7d5497f8nc4nnb1m0000gn/T/ipykernel_35157/644299150.py:1
|           `- 10.007 main  ../../../../../var/folders/_n/2hcmx0kn7d5497f8nc4nnb1m0000gn/T/ipykernel_35157/2529415599.py:67
|              `- 10.007 handle_user_request  ../../../../../var/folders/_n/2hcmx0kn7d5497f8nc4nnb1m0000gn/T/ipykernel_35157/2529415599.py:57
|                 |- 3.002 process_request_with_llm  ../../../../../var/folders/_n/2hcmx0kn7d5497f8nc4nnb1m0000gn/T/ipykernel_35157/2529415599.py:35
|                 |  `- 3.002 sleep  asyncio/tasks.py:627
|                 |        [2 frames hidden]  asyncio
|                 |           3.002 [await]  asyncio/tasks.py
|                 |- 2.002 postprocess_output  ../../../../../var/folders/_n/2hcmx0kn7d5497f8nc4nnb1m0000gn/T/ipykernel_35157/2529415599.py:42
|                 |  |- 1.001 sleep  asyncio/tasks.py:627
|                 |  |     [2 frames hidden]  asyncio
|                 |  `- 1.001 unnecesary_io_operation  ../../../../../var/folders/_n/2hcmx0kn7d5497f8nc4nnb1m0000gn/T/ipykernel_35157/2529415599.py:4
|                 |     `- 1.001 sleep  asyncio/tasks.py:627
|                 |           [2 frames hidden]  asyncio
|                 |- 2.001 fetch_user_data  ../../../../../var/folders/_n/2hcmx0kn7d5497f8nc4nnb1m0000gn/T/ipykernel_35157/2529415599.py:11
|                 |  `- 2.001 sleep  asyncio/tasks.py:627
|                 |        [2 frames hidden]  asyncio
|                 |- 2.001 store_results  ../../../../../var/folders/_n/2hcmx0kn7d5497f8nc4nnb1m0000gn/T/ipykernel_35157/2529415599.py:50
|                 |  `- 2.001 sleep  asyncio/tasks.py:627
|                 |        [2 frames hidden]  asyncio
|                 `- 1.001 preprocess_input  ../../../../../var/folders/_n/2hcmx0kn7d5497f8nc4nnb1m0000gn/T/ipykernel_35157/2529415599.py:18
|                    `- 1.001 sleep  asyncio/tasks.py:627
|                          [2 frames hidden]  asyncio
`- 1.000 KqueueSelector.select  selectors.py:553
      [2 frames hidden]  selectors, <built-in>
```

### Interpreting the output

The output is organized as a tree, with the root node representing the entry point of the application. Each node represents a function call, and the indentation level indicates the call stack depth. The leftmost column indicates the total time spent in a function and its children. The rightmost column indicates the time spent in a function alone.

In our example, we can see that the application spends most of its time fetching data for the user, making parallel API calls, and processing the request with the language model. And we can see that this time application spent in the `sleep` function, which is used to simulate I/O operations.

## Drilling down further with `yappi`

**`yappi`** is another Python profiler that supports profiling asynchronous code.

### Decorator for profiling an async function

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
