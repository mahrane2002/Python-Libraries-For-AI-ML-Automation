# asyncio — Asynchronous I/O in Python

> **Python Standard Library** · `import asyncio` · Python 3.12+
>
> The `asyncio` module provides infrastructure for writing **single-threaded concurrent** code using coroutines, multiplexing I/O access over sockets and other resources, running network clients and servers, and other related primitives.

---

## Table of Contents

- [1. What Is asyncio?](#1-what-is-asyncio)
- [2. Synchronous vs Asynchronous Execution](#2-synchronous-vs-asynchronous-execution)
- [3. Core Concepts](#3-core-concepts)
  - [3.1 The Event Loop](#31-the-event-loop)
  - [3.2 Coroutines](#32-coroutines)
  - [3.3 Awaitables](#33-awaitables)
  - [3.4 Tasks](#34-tasks)
  - [3.5 Futures](#35-futures)
- [4. Running Async Code](#4-running-async-code)
  - [4.1 asyncio.run()](#41-asynciorun)
  - [4.2 asyncio.Runner](#42-asynciorunner)
- [5. Creating and Managing Tasks](#5-creating-and-managing-tasks)
  - [5.1 asyncio.create_task()](#51-asynciocreate_task)
  - [5.2 Task Naming](#52-task-naming)
  - [5.3 Task Cancellation](#53-task-cancellation)
- [6. Gathering and Coordinating Coroutines](#6-gathering-and-coordinating-coroutines)
  - [6.1 asyncio.gather()](#61-asynciogather)
  - [6.2 asyncio.TaskGroup](#62-asynciotaskgroup)
  - [6.3 asyncio.wait()](#63-asynciowait)
  - [6.4 asyncio.wait_for()](#64-asynciowait_for)
  - [6.5 asyncio.as_completed()](#65-asyncioas_completed)
- [7. Sleeping and Timeouts](#7-sleeping-and-timeouts)
  - [7.1 asyncio.sleep()](#71-asynciosleep)
  - [7.2 asyncio.timeout()](#72-asynciotimeout)
  - [7.3 asyncio.timeout_at()](#73-asynciotimeout_at)
- [8. Synchronization Primitives](#8-synchronization-primitives)
  - [8.1 Lock](#81-lock)
  - [8.2 Event](#82-event)
  - [8.3 Semaphore](#83-semaphore)
  - [8.4 Condition](#84-condition)
  - [8.5 Barrier](#85-barrier)
- [9. Queues](#9-queues)
  - [9.1 asyncio.Queue](#91-asyncioqueue)
  - [9.2 Priority and LIFO Queues](#92-priority-and-lifo-queues)
- [10. Streams — High-Level Network I/O](#10-streams--high-level-network-io)
  - [10.1 TCP Client](#101-tcp-client)
  - [10.2 TCP Server](#102-tcp-server)
- [11. Subprocesses](#11-subprocesses)
- [12. Running Blocking Code in Async Context](#12-running-blocking-code-in-async-context)
  - [12.1 asyncio.to_thread()](#121-asyncioto_thread)
  - [12.2 loop.run_in_executor()](#122-looprun_in_executor)
- [13. Async Iterators and Generators](#13-async-iterators-and-generators)
  - [13.1 Async Iterators](#131-async-iterators)
  - [13.2 Async Generators](#132-async-generators)
  - [13.3 async for](#133-async-for)
- [14. Async Context Managers](#14-async-context-managers)
- [15. Error Handling](#15-error-handling)
  - [15.1 Exception Groups in TaskGroups](#151-exception-groups-in-taskgroups)
  - [15.2 Handling CancelledError](#152-handling-cancellederror)
  - [15.3 Unhandled Exceptions in Tasks](#153-unhandled-exceptions-in-tasks)
- [16. Debugging asyncio](#16-debugging-asyncio)
- [17. Performance Patterns](#17-performance-patterns)
  - [17.1 Limiting Concurrency](#171-limiting-concurrency)
  - [17.2 Producer-Consumer Pattern](#172-producer-consumer-pattern)
  - [17.3 Graceful Shutdown](#173-graceful-shutdown)
- [18. Common Mistakes and Pitfalls](#18-common-mistakes-and-pitfalls)
- [19. asyncio Cheat Sheet](#19-asyncio-cheat-sheet)
- [20. Summary](#20-summary)

---

## 1. What Is asyncio?

`asyncio` is a Python **standard library** module that enables **asynchronous programming** using the `async`/`await` syntax. It provides:

| Feature | Description |
|---|---|
| **Event Loop** | The central execution engine that schedules and runs coroutines |
| **Coroutines** | Functions defined with `async def` that can be paused and resumed |
| **Tasks** | Wrappers that schedule coroutines for concurrent execution |
| **Futures** | Low-level objects representing eventual results of async operations |
| **Synchronization** | Locks, events, semaphores, conditions, and barriers |
| **Queues** | Async-safe FIFO, LIFO, and priority queues |
| **Streams** | High-level API for TCP/Unix socket network I/O |
| **Subprocesses** | Async subprocess creation and management |

`asyncio` is **not** parallelism — it is **concurrency** on a single thread. It excels at **I/O-bound** workloads where the program spends time waiting for network responses, file operations, or database queries.

```
┌──────────────────────────────────────────────────────────────┐
│                      asyncio Architecture                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│   │  Coroutine  │  │  Coroutine  │  │  Coroutine  │         │
│   │   (Task A)  │  │   (Task B)  │  │   (Task C)  │         │
│   └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
│          │                │                │                 │
│          ▼                ▼                ▼                 │
│   ┌──────────────────────────────────────────────────┐       │
│   │               Event Loop (Single Thread)         │       │
│   │  ┌──────┐  ┌──────────┐  ┌───────────────────┐  │       │
│   │  │ Ready │  │ Scheduled│  │   I/O Selectors   │  │       │
│   │  │ Queue │  │  Queue   │  │ (epoll/kqueue/..) │  │       │
│   │  └──────┘  └──────────┘  └───────────────────┘  │       │
│   └──────────────────────────────────────────────────┘       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Synchronous vs Asynchronous Execution

Understanding the difference between synchronous (blocking) and asynchronous (non-blocking) execution is fundamental to understanding `asyncio`.

### Synchronous Execution (Blocking)

In synchronous code, operations execute **sequentially**. Each operation must complete before the next one starts. If an operation blocks (e.g., a network request), the entire program halts.

```
Task A:  ████████░░░░░░░░████████
Task B:  ░░░░░░░░████████░░░░░░░░████████
         ──────────────────────────────────▶ time
         (Tasks run one after another)
         Total: Sum of all task durations
```

```python
import time


def fetch_data(name: str, delay: float) -> str:
    print(f"[{name}] Starting fetch...")
    time.sleep(delay)
    print(f"[{name}] Done.")
    return f"{name}_result"


def main() -> None:
    start = time.perf_counter()

    result_a = fetch_data("API_A", 2.0)
    result_b = fetch_data("API_B", 1.5)
    result_c = fetch_data("API_C", 1.0)

    elapsed = time.perf_counter() - start
    print(f"Results: {result_a}, {result_b}, {result_c}")
    print(f"Synchronous total: {elapsed:.2f}s")  # ~4.5s


main()
```

### Asynchronous Execution (Non-Blocking)

In asynchronous code, when one coroutine awaits an I/O operation, the event loop switches to another coroutine. All I/O-bound operations overlap.

```
Task A:  ████████────────────████
Task B:  ──████████──────────████
Task C:  ────────████████────████
         ──────────────────────────▶ time
         (Tasks run concurrently, overlapping I/O waits)
         Total: Duration of the longest task
```

```python
import asyncio


async def fetch_data(name: str, delay: float) -> str:
    print(f"[{name}] Starting fetch...")
    await asyncio.sleep(delay)
    print(f"[{name}] Done.")
    return f"{name}_result"


async def main() -> None:
    start = asyncio.get_event_loop().time()

    results = await asyncio.gather(
        fetch_data("API_A", 2.0),
        fetch_data("API_B", 1.5),
        fetch_data("API_C", 1.0),
    )

    elapsed = asyncio.get_event_loop().time() - start
    print(f"Results: {results}")
    print(f"Asynchronous total: {elapsed:.2f}s")  # ~2.0s


asyncio.run(main())
```

### Comparison Summary

| Aspect | Synchronous | Asynchronous |
|---|---|---|
| **Execution model** | Sequential, blocking | Concurrent, non-blocking |
| **Thread usage** | One thread per blocking call (or blocks) | Single thread, multiplexed |
| **Best for** | CPU-bound work | I/O-bound work |
| **Syntax** | Regular `def`, regular calls | `async def`, `await` |
| **Time for N parallel I/O ops** | Sum of all durations | Duration of the longest |
| **Complexity** | Simple | Requires understanding event loop |

---

## 3. Core Concepts

### 3.1 The Event Loop

The **event loop** is the core of `asyncio`. It is a single-threaded loop that:

1. Schedules and executes coroutines
2. Handles I/O callbacks
3. Manages timers and delayed calls
4. Runs subprocesses

```
          ┌───────────────────────────────┐
          │      Event Loop Cycle         │
          │                               │
          │   ┌──────────────────────┐    │
          │   │ 1. Poll I/O events   │    │
          │   └─────────┬────────────┘    │
          │             ▼                 │
          │   ┌──────────────────────┐    │
          │   │ 2. Run ready callbacks│   │
          │   └─────────┬────────────┘    │
          │             ▼                 │
          │   ┌──────────────────────┐    │
          │   │ 3. Schedule timers   │    │
          │   └─────────┬────────────┘    │
          │             ▼                 │
          │   ┌──────────────────────┐    │
          │   │ 4. Repeat            │    │
          │   └──────────────────────┘    │
          └───────────────────────────────┘
```

In modern `asyncio` (Python 3.12+), you almost **never** interact with the event loop directly. Instead, use the high-level APIs like `asyncio.run()`, `asyncio.create_task()`, and `asyncio.gather()`.

```python
import asyncio


async def main() -> None:
    loop = asyncio.get_running_loop()
    print(f"Event loop type: {type(loop).__name__}")
    print(f"Event loop running: {loop.is_running()}")
    print(f"Event loop time: {loop.time():.4f}")


asyncio.run(main())
```

### 3.2 Coroutines

A **coroutine** is a function defined with `async def`. Calling it does **not** execute it — it returns a **coroutine object**. The coroutine only runs when it is `await`ed or scheduled as a task.

```python
import asyncio


async def greet(name: str) -> str:
    """A simple coroutine that returns a greeting."""
    await asyncio.sleep(0.1)
    return f"Hello, {name}!"


async def main() -> None:
    coro = greet("World")
    print(f"Type: {type(coro)}")  # <class 'coroutine'>

    result = await coro
    print(result)  # Hello, World!


asyncio.run(main())
```

> **Key Rule:** Every coroutine must be either `await`ed or wrapped in a `Task`. Failing to do so means the coroutine **never executes** and Python raises a `RuntimeWarning`.

### 3.3 Awaitables

An **awaitable** is any object that can be used with `await`. There are three types:

```
                  Awaitables
                      │
         ┌────────────┼────────────┐
         ▼            ▼            ▼
    Coroutines     Tasks       Futures
   (async def)  (create_task)  (low-level)
```

| Awaitable | Created by | Purpose |
|---|---|---|
| **Coroutine** | `async def` function call | Lazy — does nothing until awaited |
| **Task** | `asyncio.create_task()` | Eagerly scheduled on the event loop |
| **Future** | `loop.create_future()` | Low-level placeholder for a result |

```python
import asyncio


async def compute(x: int) -> int:
    await asyncio.sleep(0.1)
    return x * x


async def main() -> None:
    coro = compute(5)
    print(f"Coroutine is awaitable: {hasattr(coro, '__await__')}")

    task = asyncio.create_task(compute(10))
    print(f"Task is awaitable: {hasattr(task, '__await__')}")

    loop = asyncio.get_running_loop()
    future = loop.create_future()
    print(f"Future is awaitable: {hasattr(future, '__await__')}")

    result_coro = await coro
    result_task = await task
    future.set_result(42)
    result_future = await future

    print(f"Coroutine result: {result_coro}")
    print(f"Task result: {result_task}")
    print(f"Future result: {result_future}")


asyncio.run(main())
```

### 3.4 Tasks

A **Task** wraps a coroutine and schedules it for execution on the event loop **immediately**. Unlike a bare coroutine, a task starts running as soon as it is created (at the next event loop iteration).

```
   Coroutine                    Task
   ─────────                    ────
   Does NOT run                 Runs immediately
   until awaited                (scheduled on event loop)
                                
   coro = my_func()             task = create_task(my_func())
   # nothing happens            # starts running
   await coro                   await task
   # now it runs                # collects result
```

```python
import asyncio


async def worker(task_id: int, delay: float) -> str:
    print(f"  Task {task_id}: started")
    await asyncio.sleep(delay)
    print(f"  Task {task_id}: finished")
    return f"result_{task_id}"


async def main() -> None:
    print("Creating tasks...")
    task1 = asyncio.create_task(worker(1, 2.0))
    task2 = asyncio.create_task(worker(2, 1.0))
    task3 = asyncio.create_task(worker(3, 0.5))

    print("All tasks created. Awaiting results...\n")

    result1 = await task1
    result2 = await task2
    result3 = await task3

    print(f"\nResults: {result1}, {result2}, {result3}")


asyncio.run(main())
```

### 3.5 Futures

A **Future** is a low-level awaitable object representing a result that is **not yet available**. Tasks inherit from Future. In most application code, you will use Tasks rather than raw Futures.

```python
import asyncio


async def set_future_result(future: asyncio.Future, value: int) -> None:
    await asyncio.sleep(1.0)
    future.set_result(value)
    print(f"Future result set to: {value}")


async def main() -> None:
    loop = asyncio.get_running_loop()
    future = loop.create_future()

    asyncio.create_task(set_future_result(future, 42))

    print(f"Future done: {future.done()}")
    result = await future
    print(f"Future done: {future.done()}")
    print(f"Future result: {result}")


asyncio.run(main())
```

---

## 4. Running Async Code

### 4.1 asyncio.run()

`asyncio.run()` is the **primary entry point** for running async code. It creates a new event loop, runs the given coroutine, and closes the loop when done.

```python
import asyncio


async def main() -> None:
    print("Application started")
    await asyncio.sleep(0.5)
    print("Application finished")


asyncio.run(main())
```

**Signature:**

```python
asyncio.run(coro, *, debug=False)
```

| Parameter | Description |
|---|---|
| `coro` | The coroutine to run |
| `debug` | Enable debug mode for the event loop |

> **Important:** `asyncio.run()` must be called from synchronous code. It cannot be called when an event loop is already running. It always creates a **new** event loop.

### 4.2 asyncio.Runner

`asyncio.Runner` (Python 3.11+) provides a **reusable context** for running multiple top-level async functions within the same event loop.

```python
import asyncio


async def task_a() -> str:
    await asyncio.sleep(0.1)
    return "Result A"


async def task_b() -> str:
    await asyncio.sleep(0.1)
    return "Result B"


with asyncio.Runner() as runner:
    print(runner.run(task_a()))
    print(runner.run(task_b()))
```

This is useful in test frameworks and scripts where you need to run multiple independent async operations without creating a new event loop each time.

---

## 5. Creating and Managing Tasks

### 5.1 asyncio.create_task()

`asyncio.create_task()` wraps a coroutine into a Task and schedules it for execution. The coroutine begins running at the next opportunity the event loop gets.

```python
import asyncio


async def download(url: str) -> str:
    print(f"  Downloading {url}...")
    await asyncio.sleep(1.0)
    return f"data_from_{url}"


async def main() -> None:
    task1 = asyncio.create_task(download("https://api.example.com/users"))
    task2 = asyncio.create_task(download("https://api.example.com/posts"))

    data1 = await task1
    data2 = await task2
    print(f"  Downloaded: {data1}, {data2}")


asyncio.run(main())
```

### 5.2 Task Naming

Tasks can be **named** for easier debugging and logging.

```python
import asyncio


async def process_order(order_id: int) -> None:
    await asyncio.sleep(0.5)
    print(f"  Order {order_id} processed")


async def main() -> None:
    task = asyncio.create_task(
        process_order(42),
        name="process-order-42",
    )
    print(f"  Task name: {task.get_name()}")
    await task


asyncio.run(main())
```

### 5.3 Task Cancellation

Tasks can be **cancelled** using `task.cancel()`. A cancelled task raises `asyncio.CancelledError` at the next `await` point.

```python
import asyncio


async def long_running_operation() -> None:
    try:
        print("  Operation started (will run for 10 seconds)...")
        await asyncio.sleep(10.0)
        print("  Operation completed.")
    except asyncio.CancelledError:
        print("  Operation was cancelled! Cleaning up...")
        raise


async def main() -> None:
    task = asyncio.create_task(long_running_operation())

    await asyncio.sleep(1.0)
    print("  Cancelling the task...")
    task.cancel()

    try:
        await task
    except asyncio.CancelledError:
        print("  Confirmed: task is cancelled.")

    print(f"  Task cancelled: {task.cancelled()}")


asyncio.run(main())
```

> **Best Practice:** Always re-raise `CancelledError` after cleanup. Swallowing it prevents proper task cancellation propagation.

---

## 6. Gathering and Coordinating Coroutines

### 6.1 asyncio.gather()

`asyncio.gather()` runs multiple awaitables **concurrently** and returns their results as a list in the **same order** as the input.

```python
import asyncio


async def fetch_user(user_id: int) -> dict:
    await asyncio.sleep(0.5)
    return {"id": user_id, "name": f"User_{user_id}"}


async def fetch_posts(user_id: int) -> list[dict]:
    await asyncio.sleep(0.8)
    return [{"user_id": user_id, "title": f"Post by User_{user_id}"}]


async def fetch_comments(post_id: int) -> list[dict]:
    await asyncio.sleep(0.3)
    return [{"post_id": post_id, "body": "Great post!"}]


async def main() -> None:
    user, posts, comments = await asyncio.gather(
        fetch_user(1),
        fetch_posts(1),
        fetch_comments(101),
    )
    print(f"User:     {user}")
    print(f"Posts:    {posts}")
    print(f"Comments: {comments}")


asyncio.run(main())
```

**The `return_exceptions` parameter:**

When `return_exceptions=True`, exceptions are returned as values instead of being raised.

```python
import asyncio


async def succeed() -> str:
    return "success"


async def fail() -> str:
    raise ValueError("something went wrong")


async def main() -> None:
    results = await asyncio.gather(
        succeed(),
        fail(),
        succeed(),
        return_exceptions=True,
    )
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"  Task {i}: FAILED with {result!r}")
        else:
            print(f"  Task {i}: {result}")


asyncio.run(main())
```

### 6.2 asyncio.TaskGroup

`TaskGroup` (Python 3.11+) is the **modern, structured** way to manage concurrent tasks. It ensures that if any task fails, all other tasks in the group are **cancelled**.

```
              asyncio.gather()  vs  asyncio.TaskGroup
              ─────────────────     ──────────────────
              - Unstructured        - Structured concurrency
              - Errors can be       - All tasks cancelled on
                silenced              first failure
              - No cleanup          - Automatic cleanup
                guarantee             guaranteed
```

```python
import asyncio


async def fetch_resource(name: str, delay: float) -> str:
    await asyncio.sleep(delay)
    print(f"  Fetched {name}")
    return f"{name}_data"


async def main() -> None:
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(fetch_resource("users", 1.0))
        task2 = tg.create_task(fetch_resource("posts", 0.5))
        task3 = tg.create_task(fetch_resource("comments", 0.8))

    print(f"  Results: {task1.result()}, {task2.result()}, {task3.result()}")


asyncio.run(main())
```

**TaskGroup with error handling:**

```python
import asyncio


async def reliable_task(name: str) -> str:
    await asyncio.sleep(0.5)
    return f"{name}_ok"


async def failing_task() -> str:
    await asyncio.sleep(0.2)
    raise RuntimeError("Task failed!")


async def main() -> None:
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(reliable_task("A"))
            tg.create_task(failing_task())
            tg.create_task(reliable_task("B"))
    except* RuntimeError as eg:
        for exc in eg.exceptions:
            print(f"  Caught: {exc!r}")


asyncio.run(main())
```

### 6.3 asyncio.wait()

`asyncio.wait()` provides fine-grained control over when to return. Unlike `gather()`, it returns **sets** of `(done, pending)` tasks.

**Return conditions:**

| Constant | Behavior |
|---|---|
| `FIRST_COMPLETED` | Return when any task finishes |
| `FIRST_EXCEPTION` | Return when any task raises an exception |
| `ALL_COMPLETED` | Return when all tasks finish (default) |

```python
import asyncio


async def task(name: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return f"{name} complete"


async def main() -> None:
    tasks = {
        asyncio.create_task(task("Fast", 0.5), name="fast"),
        asyncio.create_task(task("Medium", 1.5), name="medium"),
        asyncio.create_task(task("Slow", 2.5), name="slow"),
    }

    done, pending = await asyncio.wait(
        tasks,
        return_when=asyncio.FIRST_COMPLETED,
    )

    print("Completed:")
    for t in done:
        print(f"  {t.get_name()}: {t.result()}")

    print(f"Still pending: {len(pending)} task(s)")

    for t in pending:
        t.cancel()

    await asyncio.gather(*pending, return_exceptions=True)


asyncio.run(main())
```

### 6.4 asyncio.wait_for()

`asyncio.wait_for()` runs a coroutine with a **timeout**. If the coroutine does not finish in time, it is cancelled and `asyncio.TimeoutError` is raised.

```python
import asyncio


async def slow_operation() -> str:
    await asyncio.sleep(5.0)
    return "done"


async def main() -> None:
    try:
        result = await asyncio.wait_for(slow_operation(), timeout=2.0)
        print(f"Result: {result}")
    except asyncio.TimeoutError:
        print("Operation timed out after 2 seconds!")


asyncio.run(main())
```

### 6.5 asyncio.as_completed()

`asyncio.as_completed()` returns an **async iterator** of awaitables that yield results in the **order they complete**, not the order they were submitted.

```python
import asyncio


async def fetch(name: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return f"{name} (took {delay}s)"


async def main() -> None:
    coros = [
        fetch("Slow", 2.0),
        fetch("Fast", 0.5),
        fetch("Medium", 1.0),
    ]

    print("Results in completion order:")
    async for coro in asyncio.as_completed(coros):
        result = await coro
        print(f"  {result}")


asyncio.run(main())
```

---

## 7. Sleeping and Timeouts

### 7.1 asyncio.sleep()

`asyncio.sleep()` suspends the current coroutine for the given number of seconds, **yielding control** to the event loop so other tasks can run.

```python
import asyncio


async def countdown(name: str, seconds: int) -> None:
    for i in range(seconds, 0, -1):
        print(f"  [{name}] {i}...")
        await asyncio.sleep(1.0)
    print(f"  [{name}] Done!")


async def main() -> None:
    await asyncio.gather(
        countdown("Timer A", 3),
        countdown("Timer B", 5),
    )


asyncio.run(main())
```

> **Important:** `asyncio.sleep(0)` yields control to the event loop without any delay. This is useful for allowing other pending tasks to run.

### 7.2 asyncio.timeout()

`asyncio.timeout()` (Python 3.11+) is an **async context manager** that enforces a timeout on a block of code.

```python
import asyncio


async def main() -> None:
    try:
        async with asyncio.timeout(2.0):
            print("Starting long operation...")
            await asyncio.sleep(5.0)
            print("This line will not execute.")
    except TimeoutError:
        print("Operation timed out!")


asyncio.run(main())
```

### 7.3 asyncio.timeout_at()

`asyncio.timeout_at()` (Python 3.11+) is similar to `asyncio.timeout()`, but accepts an **absolute** deadline based on the event loop clock.

```python
import asyncio


async def main() -> None:
    loop = asyncio.get_running_loop()
    deadline = loop.time() + 2.0

    try:
        async with asyncio.timeout_at(deadline):
            print("Working with absolute deadline...")
            await asyncio.sleep(5.0)
    except TimeoutError:
        print("Deadline exceeded!")


asyncio.run(main())
```

---

## 8. Synchronization Primitives

Even though `asyncio` is single-threaded, **synchronization primitives** are needed to coordinate access to shared resources between coroutines that yield control at `await` points.

```
┌──────────────────────────────────────────────────────┐
│            asyncio Synchronization Primitives        │
├──────────────┬───────────────────────────────────────┤
│  Lock        │  Mutual exclusion (one at a time)     │
│  Event       │  Notify coroutines of a state change  │
│  Semaphore   │  Limit concurrent access (N at a time)│
│  Condition   │  Wait for a condition to be met       │
│  Barrier     │  Synchronize N coroutines at a point  │
└──────────────┴───────────────────────────────────────┘
```

### 8.1 Lock

An `asyncio.Lock` ensures that only **one coroutine** accesses a critical section at a time.

```python
import asyncio


async def update_balance(
    lock: asyncio.Lock,
    balance: dict,
    amount: float,
    label: str,
) -> None:
    async with lock:
        current = balance["value"]
        await asyncio.sleep(0.1)  # simulate processing
        balance["value"] = current + amount
        print(f"  {label}: balance = {balance['value']:.2f}")


async def main() -> None:
    lock = asyncio.Lock()
    balance = {"value": 1000.0}

    await asyncio.gather(
        update_balance(lock, balance, -200.0, "Withdrawal"),
        update_balance(lock, balance, 500.0, "Deposit"),
        update_balance(lock, balance, -50.0, "Fee"),
    )

    print(f"  Final balance: {balance['value']:.2f}")


asyncio.run(main())
```

### 8.2 Event

An `asyncio.Event` allows one coroutine to **notify** other coroutines that something has happened.

```python
import asyncio


async def waiter(event: asyncio.Event, name: str) -> None:
    print(f"  {name}: waiting for event...")
    await event.wait()
    print(f"  {name}: event received! Proceeding.")


async def setter(event: asyncio.Event) -> None:
    print("  Setter: preparing data...")
    await asyncio.sleep(2.0)
    print("  Setter: setting event!")
    event.set()


async def main() -> None:
    event = asyncio.Event()

    await asyncio.gather(
        waiter(event, "Worker-1"),
        waiter(event, "Worker-2"),
        waiter(event, "Worker-3"),
        setter(event),
    )


asyncio.run(main())
```

### 8.3 Semaphore

An `asyncio.Semaphore` limits the number of coroutines that can access a resource **concurrently**.

```python
import asyncio


async def access_resource(
    semaphore: asyncio.Semaphore,
    resource_id: int,
) -> None:
    async with semaphore:
        print(f"  Resource {resource_id}: acquired (active)")
        await asyncio.sleep(1.0)
        print(f"  Resource {resource_id}: released")


async def main() -> None:
    semaphore = asyncio.Semaphore(3)  # max 3 concurrent

    tasks = [access_resource(semaphore, i) for i in range(10)]
    await asyncio.gather(*tasks)


asyncio.run(main())
```

### 8.4 Condition

An `asyncio.Condition` combines a lock with the ability to **wait** for a specific condition to become true.

```python
import asyncio


async def consumer(
    condition: asyncio.Condition,
    shared_data: dict,
    name: str,
) -> None:
    async with condition:
        await condition.wait_for(lambda: shared_data["ready"])
        print(f"  {name}: consumed {shared_data['item']}")


async def producer(
    condition: asyncio.Condition,
    shared_data: dict,
) -> None:
    await asyncio.sleep(1.0)
    async with condition:
        shared_data["item"] = "Premium Widget"
        shared_data["ready"] = True
        print(f"  Producer: produced {shared_data['item']}")
        condition.notify_all()


async def main() -> None:
    condition = asyncio.Condition()
    shared_data = {"ready": False, "item": None}

    await asyncio.gather(
        consumer(condition, shared_data, "Consumer-A"),
        consumer(condition, shared_data, "Consumer-B"),
        producer(condition, shared_data),
    )


asyncio.run(main())
```

### 8.5 Barrier

An `asyncio.Barrier` (Python 3.11+) blocks all participating coroutines until **all of them** have reached the barrier point.

```python
import asyncio


async def worker(barrier: asyncio.Barrier, worker_id: int) -> None:
    print(f"  Worker {worker_id}: preparing...")
    await asyncio.sleep(worker_id * 0.5)
    print(f"  Worker {worker_id}: reached barrier, waiting...")

    await barrier.wait()
    print(f"  Worker {worker_id}: barrier passed! Proceeding.")


async def main() -> None:
    num_workers = 4
    barrier = asyncio.Barrier(num_workers)

    tasks = [worker(barrier, i) for i in range(num_workers)]
    await asyncio.gather(*tasks)


asyncio.run(main())
```

---

## 9. Queues

`asyncio` provides async-safe queues for **producer-consumer** communication patterns between coroutines.

### 9.1 asyncio.Queue

`asyncio.Queue` is a FIFO queue. If the queue is full, `put()` blocks. If the queue is empty, `get()` blocks.

```python
import asyncio


async def producer(queue: asyncio.Queue, producer_id: int) -> None:
    for i in range(5):
        item = f"item-{producer_id}-{i}"
        await queue.put(item)
        print(f"  Producer {producer_id}: put {item}")
        await asyncio.sleep(0.2)


async def consumer(queue: asyncio.Queue, consumer_id: int) -> None:
    while True:
        item = await queue.get()
        print(f"  Consumer {consumer_id}: got {item}")
        await asyncio.sleep(0.5)
        queue.task_done()


async def main() -> None:
    queue: asyncio.Queue[str] = asyncio.Queue(maxsize=10)

    producers = [
        asyncio.create_task(producer(queue, i)) for i in range(2)
    ]
    consumers = [
        asyncio.create_task(consumer(queue, i)) for i in range(3)
    ]

    await asyncio.gather(*producers)
    await queue.join()

    for c in consumers:
        c.cancel()

    print(f"  All items processed. Queue size: {queue.qsize()}")


asyncio.run(main())
```

### 9.2 Priority and LIFO Queues

| Queue Type | Class | Order |
|---|---|---|
| **FIFO** | `asyncio.Queue` | First In, First Out |
| **LIFO** | `asyncio.LifoQueue` | Last In, First Out (stack) |
| **Priority** | `asyncio.PriorityQueue` | Lowest value first |

```python
import asyncio


async def main() -> None:
    pq: asyncio.PriorityQueue[tuple[int, str]] = asyncio.PriorityQueue()

    await pq.put((3, "Low priority task"))
    await pq.put((1, "High priority task"))
    await pq.put((2, "Medium priority task"))

    while not pq.empty():
        priority, item = await pq.get()
        print(f"  Priority {priority}: {item}")

    print()

    lifo: asyncio.LifoQueue[str] = asyncio.LifoQueue()

    await lifo.put("First")
    await lifo.put("Second")
    await lifo.put("Third")

    while not lifo.empty():
        item = await lifo.get()
        print(f"  LIFO: {item}")


asyncio.run(main())
```

---

## 10. Streams — High-Level Network I/O

`asyncio` streams provide a high-level API for working with **TCP** and **Unix socket** connections without dealing with protocols and transports directly.

### 10.1 TCP Client

```python
import asyncio


async def tcp_client(host: str, port: int, message: str) -> str:
    reader, writer = await asyncio.open_connection(host, port)

    print(f"  Sending: {message!r}")
    writer.write(message.encode())
    await writer.drain()

    data = await reader.read(4096)
    response = data.decode()
    print(f"  Received: {response!r}")

    writer.close()
    await writer.wait_closed()

    return response


async def main() -> None:
    try:
        response = await asyncio.wait_for(
            tcp_client("example.com", 80, "GET / HTTP/1.0\r\nHost: example.com\r\n\r\n"),
            timeout=10.0,
        )
        print(f"  Response length: {len(response)} bytes")
    except (OSError, asyncio.TimeoutError) as e:
        print(f"  Connection failed: {e}")


asyncio.run(main())
```

### 10.2 TCP Server

```python
import asyncio


async def handle_client(
    reader: asyncio.StreamReader,
    writer: asyncio.StreamWriter,
) -> None:
    addr = writer.get_extra_info("peername")
    print(f"  New connection from {addr}")

    while True:
        data = await reader.read(1024)
        if not data:
            break

        message = data.decode().strip()
        print(f"  Received from {addr}: {message!r}")

        response = f"Echo: {message}\n"
        writer.write(response.encode())
        await writer.drain()

    print(f"  Connection closed: {addr}")
    writer.close()
    await writer.wait_closed()


async def main() -> None:
    server = await asyncio.start_server(handle_client, "127.0.0.1", 8888)

    addrs = [sock.getsockname() for sock in server.sockets]
    print(f"  Server running on {addrs}")

    async with server:
        await server.serve_forever()


# asyncio.run(main())  # Uncomment to run the server
```

---

## 11. Subprocesses

`asyncio` provides an API to **create and manage subprocesses** asynchronously.

```python
import asyncio


async def run_command(cmd: str) -> tuple[str, str, int]:
    process = await asyncio.create_subprocess_shell(
        cmd,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE,
    )

    stdout, stderr = await process.communicate()

    return (
        stdout.decode().strip(),
        stderr.decode().strip(),
        process.returncode,
    )


async def main() -> None:
    commands = [
        "python --version",
        "echo Hello from asyncio subprocess",
    ]

    for cmd in commands:
        stdout, stderr, returncode = await run_command(cmd)
        print(f"  Command: {cmd}")
        print(f"  stdout:  {stdout}")
        if stderr:
            print(f"  stderr:  {stderr}")
        print(f"  Return:  {returncode}")
        print()


asyncio.run(main())
```

---

## 12. Running Blocking Code in Async Context

Sometimes you need to call **blocking** (synchronous) functions from async code. `asyncio` provides tools to run blocking code **without freezing** the event loop.

```
┌─────────────────────────────────────────────────┐
│        Running Blocking Code in asyncio         │
├─────────────────────────────────────────────────┤
│                                                 │
│  asyncio.to_thread(func)                        │
│    → Runs func in a separate thread             │
│    → Best for I/O-bound blocking code           │
│    → Python 3.9+                                │
│                                                 │
│  loop.run_in_executor(executor, func)           │
│    → Runs func in a thread or process pool      │
│    → More control over executor type            │
│    → Works with ProcessPoolExecutor for CPU ops │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 12.1 asyncio.to_thread()

`asyncio.to_thread()` runs a synchronous function in a **separate thread**, freeing the event loop.

```python
import asyncio
import time


def blocking_io_operation(name: str, duration: float) -> str:
    """Simulates a blocking I/O call (e.g., file read, DB query)."""
    print(f"  [{name}] Blocking operation started (thread)")
    time.sleep(duration)
    print(f"  [{name}] Blocking operation finished")
    return f"{name}_result"


async def main() -> None:
    start = asyncio.get_event_loop().time()

    results = await asyncio.gather(
        asyncio.to_thread(blocking_io_operation, "DB_Query", 2.0),
        asyncio.to_thread(blocking_io_operation, "File_Read", 1.5),
        asyncio.to_thread(blocking_io_operation, "API_Call", 1.0),
    )

    elapsed = asyncio.get_event_loop().time() - start
    print(f"  Results: {results}")
    print(f"  Total time: {elapsed:.2f}s")  # ~2.0s, not ~4.5s


asyncio.run(main())
```

### 12.2 loop.run_in_executor()

`loop.run_in_executor()` provides more control, allowing you to use a **custom executor** (thread pool or process pool).

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor
import math


def cpu_intensive(n: int) -> float:
    """CPU-bound work that benefits from a process pool."""
    return sum(math.sqrt(i) for i in range(n))


def io_blocking(name: str) -> str:
    """I/O-bound blocking work for a thread pool."""
    import time
    time.sleep(1.0)
    return f"{name}_done"


async def main() -> None:
    loop = asyncio.get_running_loop()

    with ThreadPoolExecutor(max_workers=4) as thread_pool:
        result = await loop.run_in_executor(
            thread_pool,
            io_blocking,
            "download",
        )
        print(f"  Thread pool result: {result}")

    with ProcessPoolExecutor(max_workers=2) as process_pool:
        result = await loop.run_in_executor(
            process_pool,
            cpu_intensive,
            10_000_000,
        )
        print(f"  Process pool result: {result:.2f}")


asyncio.run(main())
```

---

## 13. Async Iterators and Generators

### 13.1 Async Iterators

An async iterator implements `__aiter__()` and `__anext__()` methods.

```python
import asyncio


class AsyncCountdown:
    """Async iterator that counts down from a given number."""

    def __init__(self, start: int) -> None:
        self.current = start

    def __aiter__(self):
        return self

    async def __anext__(self) -> int:
        if self.current <= 0:
            raise StopAsyncIteration
        self.current -= 1
        await asyncio.sleep(0.3)
        return self.current + 1


async def main() -> None:
    async for number in AsyncCountdown(5):
        print(f"  {number}")


asyncio.run(main())
```

### 13.2 Async Generators

Async generators use `async def` with `yield` to produce values asynchronously.

```python
import asyncio


async def fetch_pages(total_pages: int):
    """Simulate fetching pages from an API with pagination."""
    for page in range(1, total_pages + 1):
        await asyncio.sleep(0.5)  # simulate network delay
        data = {"page": page, "items": [f"item_{page}_{i}" for i in range(3)]}
        yield data


async def main() -> None:
    async for page_data in fetch_pages(4):
        print(f"  Page {page_data['page']}: {page_data['items']}")


asyncio.run(main())
```

### 13.3 async for

The `async for` statement iterates over any **async iterable** (an object with `__aiter__` method).

```python
import asyncio


async def generate_events():
    events = ["user_login", "page_view", "purchase", "user_logout"]
    for event in events:
        await asyncio.sleep(0.3)
        yield {"type": event, "timestamp": asyncio.get_event_loop().time()}


async def main() -> None:
    print("Processing events:")
    async for event in generate_events():
        print(f"  Event: {event['type']}")


asyncio.run(main())
```

---

## 14. Async Context Managers

Async context managers implement `__aenter__()` and `__aexit__()` and are used with `async with`.

```python
import asyncio


class AsyncDatabaseConnection:
    """Simulates an async database connection lifecycle."""

    def __init__(self, db_name: str) -> None:
        self.db_name = db_name
        self.connected = False

    async def __aenter__(self):
        print(f"  Connecting to '{self.db_name}'...")
        await asyncio.sleep(0.5)
        self.connected = True
        print(f"  Connected to '{self.db_name}'.")
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb) -> None:
        print(f"  Closing connection to '{self.db_name}'...")
        await asyncio.sleep(0.2)
        self.connected = False
        print(f"  Connection closed.")

    async def query(self, sql: str) -> list[dict]:
        if not self.connected:
            raise RuntimeError("Not connected")
        await asyncio.sleep(0.3)
        return [{"sql": sql, "result": "mock_data"}]


async def main() -> None:
    async with AsyncDatabaseConnection("mydb") as db:
        result = await db.query("SELECT * FROM users")
        print(f"  Query result: {result}")


asyncio.run(main())
```

You can also create async context managers using `contextlib.asynccontextmanager`:

```python
import asyncio
from contextlib import asynccontextmanager


@asynccontextmanager
async def managed_resource(name: str):
    print(f"  Acquiring {name}...")
    await asyncio.sleep(0.3)
    try:
        yield name
    finally:
        print(f"  Releasing {name}...")
        await asyncio.sleep(0.2)


async def main() -> None:
    async with managed_resource("GPU_Instance") as resource:
        print(f"  Using resource: {resource}")
        await asyncio.sleep(0.5)


asyncio.run(main())
```

---

## 15. Error Handling

### 15.1 Exception Groups in TaskGroups

When multiple tasks in a `TaskGroup` fail, their exceptions are collected into an `ExceptionGroup` (Python 3.11+). Use `except*` syntax to handle them.

```python
import asyncio


async def task_ok(name: str) -> str:
    await asyncio.sleep(0.1)
    return f"{name}_ok"


async def task_value_error() -> None:
    await asyncio.sleep(0.2)
    raise ValueError("Invalid value")


async def task_runtime_error() -> None:
    await asyncio.sleep(0.3)
    raise RuntimeError("Runtime failure")


async def main() -> None:
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(task_ok("A"))
            tg.create_task(task_value_error())
            tg.create_task(task_runtime_error())
    except* ValueError as eg:
        for exc in eg.exceptions:
            print(f"  ValueError caught: {exc}")
    except* RuntimeError as eg:
        for exc in eg.exceptions:
            print(f"  RuntimeError caught: {exc}")


asyncio.run(main())
```

### 15.2 Handling CancelledError

`CancelledError` is a subclass of `BaseException` (not `Exception`). This means a bare `except Exception` will **not** catch it — which is the correct behavior.

```python
import asyncio


async def cancellable_work() -> None:
    try:
        while True:
            print("  Working...")
            await asyncio.sleep(1.0)
    except asyncio.CancelledError:
        print("  Cancellation received. Running cleanup...")
        await asyncio.sleep(0.1)  # cleanup work
        print("  Cleanup complete.")
        raise  # always re-raise


async def main() -> None:
    task = asyncio.create_task(cancellable_work())
    await asyncio.sleep(2.5)
    task.cancel()

    try:
        await task
    except asyncio.CancelledError:
        print("  Task successfully cancelled.")


asyncio.run(main())
```

### 15.3 Unhandled Exceptions in Tasks

If a Task raises an exception and no one awaits it, the exception is **lost silently** (with a warning). Always await your tasks or use a `done` callback.

```python
import asyncio


async def risky_task() -> None:
    raise RuntimeError("Unhandled error!")


async def main() -> None:
    task = asyncio.create_task(risky_task(), name="risky")

    task.add_done_callback(lambda t: handle_task_result(t))

    await asyncio.sleep(1.0)


def handle_task_result(task: asyncio.Task) -> None:
    if task.cancelled():
        print(f"  Task '{task.get_name()}' was cancelled")
    elif task.exception():
        print(f"  Task '{task.get_name()}' failed: {task.exception()!r}")
    else:
        print(f"  Task '{task.get_name()}' succeeded: {task.result()}")


asyncio.run(main())
```

---

## 16. Debugging asyncio

`asyncio` has a built-in **debug mode** that provides warnings for common mistakes like:

- Coroutines that were never awaited
- Callbacks that take too long to execute
- Resource leaks

### Enabling Debug Mode

**Method 1: Via `asyncio.run()`**

```python
import asyncio


async def main() -> None:
    print(f"  Debug mode: {asyncio.get_event_loop().get_debug()}")
    await asyncio.sleep(0.1)


asyncio.run(main(), debug=True)
```

**Method 2: Via environment variable**

```python
# Set PYTHONASYNCIODEBUG=1 before running:
# Windows:   set PYTHONASYNCIODEBUG=1
# Linux/Mac: export PYTHONASYNCIODEBUG=1

import asyncio


async def main() -> None:
    loop = asyncio.get_running_loop()
    print(f"  Debug mode: {loop.get_debug()}")


asyncio.run(main())
```

### Inspecting Running Tasks

```python
import asyncio


async def background_work(task_id: int) -> None:
    await asyncio.sleep(10.0)


async def main() -> None:
    tasks = [
        asyncio.create_task(background_work(i), name=f"worker-{i}")
        for i in range(5)
    ]

    await asyncio.sleep(0.1)

    all_tasks = asyncio.all_tasks()
    print(f"  Total running tasks: {len(all_tasks)}")
    for task in sorted(all_tasks, key=lambda t: t.get_name()):
        print(f"    {task.get_name()}: done={task.done()}")

    current = asyncio.current_task()
    print(f"  Current task: {current.get_name()}")

    for task in tasks:
        task.cancel()
    await asyncio.gather(*tasks, return_exceptions=True)


asyncio.run(main())
```

---

## 17. Performance Patterns

### 17.1 Limiting Concurrency

Use a `Semaphore` to limit how many coroutines run concurrently. This is essential when hitting rate-limited APIs or databases.

```python
import asyncio


async def fetch_url(
    semaphore: asyncio.Semaphore,
    url: str,
    session_id: int,
) -> dict:
    async with semaphore:
        print(f"  [{session_id}] Fetching {url}...")
        await asyncio.sleep(1.0)  # simulate network request
        return {"url": url, "status": 200}


async def main() -> None:
    max_concurrent = 5
    semaphore = asyncio.Semaphore(max_concurrent)

    urls = [f"https://api.example.com/page/{i}" for i in range(20)]

    tasks = [
        fetch_url(semaphore, url, i)
        for i, url in enumerate(urls)
    ]

    results = await asyncio.gather(*tasks)
    print(f"\n  Total fetched: {len(results)}")
    print(f"  All status 200: {all(r['status'] == 200 for r in results)}")


asyncio.run(main())
```

### 17.2 Producer-Consumer Pattern

A robust producer-consumer pattern using `asyncio.Queue` with multiple producers and consumers.

```python
import asyncio
import random


async def producer(
    queue: asyncio.Queue,
    producer_id: int,
    num_items: int,
) -> None:
    for i in range(num_items):
        item = {"producer": producer_id, "item": i, "value": random.randint(1, 100)}
        await queue.put(item)
        print(f"  Producer-{producer_id}: produced item {i}")
        await asyncio.sleep(random.uniform(0.1, 0.5))

    print(f"  Producer-{producer_id}: finished")


async def consumer(
    queue: asyncio.Queue,
    consumer_id: int,
) -> int:
    processed = 0
    while True:
        try:
            item = await asyncio.wait_for(queue.get(), timeout=2.0)
        except asyncio.TimeoutError:
            print(f"  Consumer-{consumer_id}: timed out, stopping")
            break

        print(f"  Consumer-{consumer_id}: processing {item}")
        await asyncio.sleep(random.uniform(0.2, 0.8))
        queue.task_done()
        processed += 1

    return processed


async def main() -> None:
    queue: asyncio.Queue[dict] = asyncio.Queue(maxsize=20)

    producers = [
        asyncio.create_task(producer(queue, i, 5))
        for i in range(3)
    ]
    consumers = [
        asyncio.create_task(consumer(queue, i))
        for i in range(2)
    ]

    await asyncio.gather(*producers)
    await queue.join()

    for c in consumers:
        c.cancel()

    results = await asyncio.gather(*consumers, return_exceptions=True)
    total_processed = sum(r for r in results if isinstance(r, int))
    print(f"\n  Total items processed: {total_processed}")


asyncio.run(main())
```

### 17.3 Graceful Shutdown

A pattern for shutting down an async application gracefully, ensuring all tasks are properly cancelled and cleaned up.

```python
import asyncio
import signal


async def worker(worker_id: int) -> None:
    try:
        while True:
            print(f"  Worker-{worker_id}: heartbeat")
            await asyncio.sleep(2.0)
    except asyncio.CancelledError:
        print(f"  Worker-{worker_id}: shutting down gracefully...")
        await asyncio.sleep(0.1)
        print(f"  Worker-{worker_id}: cleanup complete")
        raise


async def shutdown(loop: asyncio.AbstractEventLoop) -> None:
    print("\n  Received shutdown signal...")
    tasks = [
        t for t in asyncio.all_tasks()
        if t is not asyncio.current_task()
    ]
    print(f"  Cancelling {len(tasks)} outstanding tasks...")

    for task in tasks:
        task.cancel()

    results = await asyncio.gather(*tasks, return_exceptions=True)

    for result in results:
        if isinstance(result, Exception) and not isinstance(
            result, asyncio.CancelledError
        ):
            print(f"  Exception during shutdown: {result!r}")

    loop.stop()
    print("  Shutdown complete.")


async def main() -> None:
    loop = asyncio.get_running_loop()

    # Register signal handlers (Unix only)
    # for sig in (signal.SIGTERM, signal.SIGINT):
    #     loop.add_signal_handler(
    #         sig,
    #         lambda: asyncio.create_task(shutdown(loop)),
    #     )

    workers = [
        asyncio.create_task(worker(i), name=f"worker-{i}")
        for i in range(3)
    ]

    await asyncio.sleep(5.0)

    await shutdown(loop)


asyncio.run(main())
```

---

## 18. Common Mistakes and Pitfalls

### ❌ Mistake 1: Forgetting to `await` a coroutine

```python
import asyncio


async def compute() -> int:
    return 42


async def wrong() -> None:
    result = compute()  # Returns a coroutine object, NOT 42
    print(type(result))  # <class 'coroutine'>


async def correct() -> None:
    result = await compute()  # Properly awaits the coroutine
    print(result)  # 42


asyncio.run(correct())
```

### ❌ Mistake 2: Calling blocking code directly in async functions

```python
import asyncio
import time


async def wrong() -> None:
    time.sleep(5)  # BLOCKS the entire event loop for 5 seconds!


async def correct() -> None:
    await asyncio.sleep(5)  # Yields control — other tasks can run


async def correct_with_blocking_lib() -> None:
    await asyncio.to_thread(time.sleep, 5)  # Runs in a thread — event loop stays free


asyncio.run(correct())
```

### ❌ Mistake 3: Creating tasks without holding a reference

```python
import asyncio


async def background_job() -> None:
    await asyncio.sleep(1.0)
    print("Job done!")  # May never print if task is garbage collected


async def wrong() -> None:
    asyncio.create_task(background_job())  # No reference kept!
    # Task may be garbage collected before it finishes


async def correct() -> None:
    task = asyncio.create_task(background_job())  # Keep a reference
    await task  # Ensure it completes


asyncio.run(correct())
```

### ❌ Mistake 4: Swallowing CancelledError

```python
import asyncio


async def wrong() -> None:
    try:
        await asyncio.sleep(10.0)
    except asyncio.CancelledError:
        print("Cancelled")
        # ERROR: Not re-raising — task appears to complete normally


async def correct() -> None:
    try:
        await asyncio.sleep(10.0)
    except asyncio.CancelledError:
        print("Cancelled — cleaning up")
        raise  # Always re-raise to propagate cancellation


asyncio.run(correct())
```

### ❌ Mistake 5: Using `asyncio.gather()` without error handling

```python
import asyncio


async def might_fail(n: int) -> int:
    if n == 3:
        raise ValueError(f"Bad value: {n}")
    return n * 10


async def wrong() -> None:
    # If any task fails, all results are lost
    results = await asyncio.gather(
        might_fail(1), might_fail(2), might_fail(3),
    )


async def correct_option_1() -> None:
    results = await asyncio.gather(
        might_fail(1), might_fail(2), might_fail(3),
        return_exceptions=True,
    )
    for r in results:
        if isinstance(r, Exception):
            print(f"  Error: {r}")
        else:
            print(f"  Result: {r}")


async def correct_option_2() -> None:
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(might_fail(1))
            tg.create_task(might_fail(2))
            tg.create_task(might_fail(3))
    except* ValueError as eg:
        for exc in eg.exceptions:
            print(f"  Caught: {exc}")


asyncio.run(correct_option_1())
```

### Summary of Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| Forgetting `await` | Coroutine never runs | Always `await` or wrap in `create_task()` |
| Blocking calls in async | Event loop freezes | Use `asyncio.to_thread()` or `run_in_executor()` |
| No task reference | Task garbage collected | Store task references in a variable or collection |
| Swallowing `CancelledError` | Cancellation breaks | Always `raise` after cleanup |
| No error handling in `gather` | All results lost on error | Use `return_exceptions=True` or `TaskGroup` |

---

## 19. asyncio Cheat Sheet

### Entry Points

```python
import asyncio

# Run a single coroutine (creates + closes event loop)
asyncio.run(main())

# Run multiple coroutines on the same loop
with asyncio.Runner() as runner:
    runner.run(coro1())
    runner.run(coro2())
```

### Creating and Running Tasks

```python
# Create a task (starts immediately)
task = asyncio.create_task(my_coroutine(), name="my-task")

# Wait for a task
result = await task

# Cancel a task
task.cancel()

# Check task state
task.done()        # True if finished (success, error, or cancelled)
task.cancelled()   # True if cancelled
task.exception()   # Returns exception or None
task.result()      # Returns result (raises if error/cancelled)
```

### Coordination

```python
# Run concurrently, collect results in order
results = await asyncio.gather(coro1(), coro2(), coro3())

# Structured concurrency (cancels all on failure)
async with asyncio.TaskGroup() as tg:
    task1 = tg.create_task(coro1())
    task2 = tg.create_task(coro2())

# Wait for first/all/exception
done, pending = await asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED)

# Process results as they complete
async for coro in asyncio.as_completed(coros):
    result = await coro
```

### Timeouts

```python
# Timeout on a single coroutine
result = await asyncio.wait_for(coro(), timeout=5.0)

# Timeout context manager
async with asyncio.timeout(5.0):
    await long_operation()

# Absolute deadline
async with asyncio.timeout_at(loop.time() + 5.0):
    await long_operation()
```

### Blocking → Async Bridge

```python
# Run blocking function in a thread
result = await asyncio.to_thread(blocking_func, arg1, arg2)

# Run in a custom executor
result = await loop.run_in_executor(executor, blocking_func, arg1)
```

### Synchronization

```python
# Lock
async with asyncio.Lock():
    ...  # exclusive access

# Semaphore (limit concurrency)
async with asyncio.Semaphore(10):
    ...  # max 10 concurrent

# Event (signal between coroutines)
event = asyncio.Event()
await event.wait()   # blocks until set
event.set()          # unblocks all waiters
event.clear()        # reset

# Queue
queue = asyncio.Queue(maxsize=100)
await queue.put(item)
item = await queue.get()
queue.task_done()
await queue.join()  # blocks until all items processed
```

### Key API Reference

| Function / Class | Purpose |
|---|---|
| `asyncio.run(coro)` | Run top-level coroutine |
| `asyncio.create_task(coro)` | Schedule coroutine as a Task |
| `asyncio.gather(*coros)` | Run awaitables concurrently |
| `asyncio.TaskGroup()` | Structured concurrent execution |
| `asyncio.wait(tasks)` | Wait with fine-grained control |
| `asyncio.wait_for(coro, timeout)` | Await with timeout |
| `asyncio.as_completed(coros)` | Iterate results by completion order |
| `asyncio.sleep(seconds)` | Non-blocking sleep |
| `asyncio.timeout(seconds)` | Timeout context manager |
| `asyncio.to_thread(func)` | Run blocking func in thread |
| `asyncio.Queue(maxsize)` | Async-safe FIFO queue |
| `asyncio.Lock()` | Mutual exclusion lock |
| `asyncio.Semaphore(n)` | Limit concurrent access |
| `asyncio.Event()` | Signal between coroutines |
| `asyncio.Condition()` | Wait for condition |
| `asyncio.Barrier(n)` | Sync N coroutines at a point |
| `asyncio.all_tasks()` | Get all running tasks |
| `asyncio.current_task()` | Get the current task |

---

## 20. Summary

`asyncio` is the foundation of asynchronous programming in Python. It enables efficient, concurrent I/O-bound operations on a single thread using coroutines, tasks, and an event loop.

**When to use `asyncio`:**

- Web scraping and HTTP clients
- API servers and microservices
- Database access with async drivers
- Chat applications and WebSocket servers
- File I/O orchestration
- Any I/O-bound workload with many concurrent operations

**When NOT to use `asyncio`:**

- CPU-bound computation (use `multiprocessing` instead)
- Simple sequential scripts with no concurrency needs
- When all third-party libraries are synchronous-only

**Key takeaways:**

1. Use `async def` and `await` to define and call coroutines
2. Use `asyncio.run()` as the single entry point
3. Use `asyncio.create_task()` to run coroutines concurrently
4. Use `asyncio.TaskGroup` (preferred) or `asyncio.gather()` to coordinate
5. Never call blocking functions directly — use `asyncio.to_thread()`
6. Always handle errors and cancellation properly
7. Use synchronization primitives when coroutines share mutable state

---

> **Python Version:** 3.12+ · **Module:** `asyncio` (standard library) · **Documentation:** [docs.python.org/3/library/asyncio.html](https://docs.python.org/3/library/asyncio.html)
