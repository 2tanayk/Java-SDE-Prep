# `ExecutorService` & Thread Pools

## Why Thread Pools?

Creating a new thread for every task does not scale well. Threads have memory, scheduling, and lifecycle costs, and too many threads can overwhelm an application.

Instead of creating a new thread for every task, keep a controlled number of worker threads and reuse them.

## Thread Pool

A thread pool is a collection of reusable worker threads that execute submitted tasks.

```text
Tasks → Queue → Worker Threads
```

When a worker finishes one task, it can execute another instead of creating a new thread.

## `ExecutorService`

Java provides `ExecutorService` for managing task execution and the executor lifecycle.

```java
ExecutorService executor =
        Executors.newFixedThreadPool(4);
```

Instead of manually creating threads, submit work:

```java
executor.submit(task);
```

The executor assigns the task to an available worker.

## Why This Is Useful Even If the Caller Eventually Waits

Submitting work to an executor is asynchronous with respect to the submitting thread, even if that thread later calls `Future.get()`.

```java
Future<?> a = executor.submit(() -> doTaskA());
Future<?> b = executor.submit(() -> doTaskB());

a.get();
b.get();
```

With two worker threads, Task A and Task B can execute concurrently:

```text
Worker 1:  [──── Task A ────]
Worker 2:  [──── Task B ────]

Main:     submit A → submit B → WAIT
```

`get()` makes the calling thread wait for the result, but it does not make the tasks execute sequentially.

If each independent task takes about 5 seconds:

- Sequential execution: about 10 seconds.
- Two independent tasks on two workers: about 5 seconds while the caller waits for completion.

This is a common backend pattern: a request can remain synchronous from the caller's perspective while independent work inside the request executes concurrently.

## `Executor` vs `ExecutorService`

`Executor` is the basic execution abstraction:

```java
void execute(Runnable command);
```

It essentially means: execute this task.

`ExecutorService` extends this concept with richer task submission and lifecycle management, including `submit()` and `shutdown()`.

## `execute()` vs `submit()`

### `execute()`

```java
executor.execute(() -> doWork());
```

Takes a `Runnable` and does not return a `Future`.

### `submit()`

```java
Future<?> future =
        executor.submit(() -> doWork());
```

Can accept `Runnable` or `Callable` and returns a `Future`.

```text
execute() → run Runnable → no Future
submit()  → submit task → Future returned
```

## `Callable`

`Runnable` represents work without a returned result.

`Callable<T>` represents work that can return a result:

```java
Callable<Integer> task = () -> 42;
```

Submit it:

```java
Future<Integer> future =
        executor.submit(task);
```

Conceptually:

```text
Thread Pool → Callable → result → Future<Integer>
```

## `Future`

A `Future` represents the eventual result of an asynchronous computation.

```java
Future<Integer> future =
        executor.submit(() -> 42);
```

At submission time, the result may not be ready. The `Future` is a handle through which the result can later be obtained.

```java
Integer result = future.get();
```

### `Future.get()` Can Block

If the task is still running, `get()` waits:

```text
Main Thread
    ↓
future.get()
    │
    │ WAIT
    ↓
Worker finishes
    ↓
returns result
```

So executor submission can be asynchronous even though calling `get()` eventually makes the caller wait.

## `shutdown()`

```java
executor.shutdown();
```

Initiates a graceful shutdown:

- No new tasks are accepted.
- Already-submitted tasks are allowed to finish.

## `shutdownNow()`

```java
executor.shutdownNow();
```

Attempts to stop actively executing tasks and returns tasks that were waiting in the queue.

It does not magically kill arbitrary Java code; task interruption needs to be handled appropriately.

```text
shutdown()    → graceful shutdown → finish submitted tasks
shutdownNow() → attempt to stop ongoing work + return queued tasks
```

## Fixed Thread Pool

```java
Executors.newFixedThreadPool(4);
```

Maintains a fixed number of worker threads.

If there are four workers and 100 tasks:

```text
4 tasks → executing
remaining tasks → queued/waiting
```

This gives controlled concurrency at the worker-thread level.

## Cached Thread Pool

```java
Executors.newCachedThreadPool();
```

Creates and reuses threads dynamically as needed. Unlike a fixed pool, the number of threads is not fixed.

It can potentially create many threads if a large amount of work is submitted, so it should not be treated as automatically better or safer.

For current preparation, understand the basic distinction:

```text
Fixed pool  → fixed number of worker threads
Cached pool → threads created/reused dynamically
```

## Backend Mental Model

A request can remain synchronous from the caller's perspective while independent work executes concurrently in a thread pool.

```text
Sequential:
Request → A → B → C → response

Concurrent:
              ┌→ A ─┐
Request → pool ├→ B ─┤→ response
              └→ C ─┘
```

The caller may still wait for all required results, but the independent work can overlap.

## Core Mental Model

```text
                 ExecutorService
                       │
                       ↓
                  Thread Pool
             ┌─────┬─────┬─────┐
             │ T1  │ T2  │ T3  │
             └─────┴─────┴─────┘
                  ↑
                  │
               Task Queue
                  ↑
                  │
             submit(task)
```

```text
Runnable      → "Do this"
Callable<T>   → "Do this and give me T"
Future<T>     → "I'll eventually give you T"
ExecutorService → "I'll manage the threads executing those tasks"
```

## Interview-Level Takeaways

- Creating a new thread for every task does not scale well.
- A thread pool reuses a controlled number of worker threads.
- `ExecutorService` manages task execution and executor lifecycle.
- `execute()` takes a `Runnable` and does not return a `Future`.
- `submit()` can take `Runnable` or `Callable` and returns a `Future`.
- `Callable<T>` can return a result.
- `Future.get()` can block until the result is available.
- Submitting tasks can be asynchronous even if the caller later waits for their results.
- `shutdown()` allows submitted tasks to finish while rejecting new tasks.
- `shutdownNow()` attempts to stop ongoing work and returns queued tasks.
- A fixed thread pool has a fixed number of workers; excess tasks wait in the queue.
