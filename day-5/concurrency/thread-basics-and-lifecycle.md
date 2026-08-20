# Thread Basics & Lifecycle

## What Is a Thread?

A **thread** is an independent path of execution within a process.

A running Java application is a process, and that process can contain multiple threads:

```text
Spring Boot Process
│
├── Thread 1
├── Thread 2
├── Thread 3
└── Thread 4
```

Threads allow multiple tasks to make progress concurrently.

This is particularly useful for I/O-bound work such as:

- Database calls
- HTTP requests
- File I/O
- Network communication

## Process vs Thread

A **process** is a running instance of a program and has its own resources/address space.

A **thread** is an execution path within a process. Threads in the same process share the process's heap/resources while each thread has its own execution stack.

## Concurrency vs Parallelism

**Concurrency** means multiple tasks are in progress during overlapping periods. They can be interleaved even on a single CPU core.

**Parallelism** means multiple tasks are actually executing simultaneously, typically on different CPU cores.

```text
Concurrency (one core):
Task A ███     ███
Task B    ███     ███

Parallelism (multiple cores):
Core 1: Task A █████████
Core 2: Task B █████████
```

Mental model:

> Concurrency = dealing with multiple tasks at once.
>
> Parallelism = executing multiple tasks simultaneously.

## Creating a Thread

The basic approach is:

```java
Thread thread = new Thread(() -> {
    System.out.println("Hello from another thread");
});

thread.start();
```

The lambda represents the work that the new thread should execute.

## `start()` vs `run()`

This is a classic interview question.

```java
Thread thread = new Thread(() -> {
    System.out.println("Hello");
});

thread.start();
```

`start()` asks the JVM to start a new thread, which will eventually invoke `run()` on that new thread.

By contrast:

```java
thread.run();
```

is simply a normal method call. It does **not** start a new thread; the code executes synchronously on the current thread.

Mental model:

```text
thread.start()
    → new thread
    → run()

thread.run()
    → normal method call
    → current thread
```

### Interview Answer

> `start()` asks the JVM to create and schedule a new thread, which eventually invokes `run()`. Calling `run()` directly is just a normal synchronous method invocation on the current thread.

## `Runnable`

`Runnable` represents a task that can be executed:

```java
@FunctionalInterface
public interface Runnable {
    void run();
}
```

Example:

```java
Runnable task = () -> {
    System.out.println("Doing work");
};

Thread thread = new Thread(task);
thread.start();
```

A useful conceptual distinction is:

```text
Runnable
   ↓
"What work should be done?"

Thread
   ↓
"Who executes that work?"
```

This separation becomes especially useful with `ExecutorService` and thread pools.

## Extending `Thread`

It is also possible to extend `Thread`:

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Working");
    }
}
```

Then:

```java
MyThread thread = new MyThread();
thread.start();
```

However, separating the task (`Runnable`) from the execution mechanism (`Thread`/executor) is generally more flexible.

## `Callable`

`Runnable` does not return a result and does not directly declare checked exceptions:

```java
void run();
```

`Callable<T>` represents a task that can return a result:

```java
Callable<Integer> task = () -> {
    return 42;
};
```

Conceptually:

```text
Runnable
   ↓
do work
   ↓
no result

Callable<T>
   ↓
do work
   ↓
return T
```

`Callable` becomes particularly useful with `ExecutorService` and `Future`.

## Thread Lifecycle / States

Java exposes these `Thread.State` values:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

### NEW

The thread has been created but not started:

```java
Thread t = new Thread(task);
```

Conceptually:

```text
NEW
```

Calling `start()` moves it toward `RUNNABLE`.

### RUNNABLE

The thread is eligible to run. Java's `RUNNABLE` state includes threads that are actually running and threads that are ready to run but waiting for CPU time.

```text
NEW
 ↓
start()
 ↓
RUNNABLE
```

### BLOCKED

The thread is waiting to acquire a monitor lock, commonly because another thread owns a `synchronized` lock it needs.

```text
Thread A → owns lock
Thread B → wants same lock
Thread B → BLOCKED
```

### WAITING

The thread is waiting indefinitely for another thread/action. Examples include `Object.wait()` and `Thread.join()` without a timeout.

### TIMED_WAITING

The thread is waiting for a specified amount of time. A common example is:

```java
Thread.sleep(5000);
```

### TERMINATED

The thread has finished executing. Once terminated, a thread cannot simply be restarted.

Conceptually:

```text
RUNNABLE
    ↓
run() completes
    ↓
TERMINATED
```

## `sleep()`

```java
Thread.sleep(1000);
```

Pauses the **current thread** for approximately the specified duration.

Important:

> `sleep()` does **not** release a lock currently held by the thread.

`Thread.sleep(...)` can throw `InterruptedException`, which is a checked exception.

## `join()`

`join()` makes the current thread wait for another thread to finish.

```java
Thread worker = new Thread(() -> {
    doWork();
});

worker.start();
worker.join();

System.out.println("Worker finished");
```

Conceptually:

```text
Main thread
    │
    ├── start worker
    │
    ↓
 wait for worker
    │
    ↓
worker terminates
    │
    ↓
main continues
```

Without `join()`, the main thread can continue independently after starting the worker.

## Complete Example

```java
public class Main {

    public static void main(String[] args)
            throws InterruptedException {

        Runnable task = () -> {
            System.out.println(
                "Running on: " +
                Thread.currentThread().getName()
            );
        };

        Thread worker = new Thread(task);

        System.out.println(worker.getState());
        // NEW

        worker.start();

        System.out.println(worker.getState());
        // RUNNABLE (usually)

        worker.join();

        System.out.println(worker.getState());
        // TERMINATED
    }
}
```

The conceptual lifecycle is:

```text
Thread created
      ↓
     NEW
      ↓
   start()
      ↓
   RUNNABLE
      ↓
    run()
      ↓
 TERMINATED
```

## Important Practical Note

Although Java allows manually creating threads, real backend applications generally should not create a brand-new thread for every task. Thread creation and scheduling have costs, and too many threads can overwhelm an application.

This is why Java provides **thread pools and `ExecutorService`**, which will be covered next.

## Interview-Level Takeaways

- A thread is an independent path of execution within a process.
- Threads in the same process share process resources/heap while each has its own stack.
- Concurrency and parallelism are related but not identical.
- `start()` starts a new thread; calling `run()` directly does not.
- `Runnable` represents work; `Thread` represents a basic execution mechanism.
- `Callable<T>` represents work that can return a result.
- Know the Java thread states: `NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING`, `TERMINATED`.
- `sleep()` pauses the current thread and does not release locks it holds.
- `join()` makes the current thread wait for another thread to finish.
- Manually creating threads for every task does not scale well; thread pools and `ExecutorService` are the normal application-level approach.
