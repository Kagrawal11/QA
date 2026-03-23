# 🧵 Part 2: Multithreading (Q47–Q63)

---

## Q47. Thread Lifecycle (Detailed)

```
                    ┌────────────┐
         start()    │            │
   NEW ──────────→ RUNNABLE ────┤
                    │   ↑        │ Thread Scheduler
                    │   │        │ picks thread
                    │   │        ↓
                    │   │     RUNNING
                    │   │      │  │
                    │   │      │  └──→ TERMINATED (run() completes / exception)
                    │   │      │
                    │   │      │ wait()/sleep()/join()/lock
                    │   │      ↓
                    │   └── BLOCKED / WAITING / TIMED_WAITING
                    │         (notify()/notifyAll()/sleep expired/lock acquired)
                    └────────────┘
```

| State | Trigger | Description |
|-------|---------|-------------|
| `NEW` | `new Thread()` | Created, not started |
| `RUNNABLE` | `start()` | Ready, waiting for CPU |
| `RUNNING` | CPU scheduler | Executing `run()` |
| `BLOCKED` | `synchronized` | Waiting for monitor lock |
| `WAITING` | `wait()`, `join()` | Indefinite wait |
| `TIMED_WAITING` | `sleep(ms)`, `wait(ms)` | Wait with timeout |
| `TERMINATED` | `run()` completes | Dead — cannot restart |

---

## Q48. Runnable vs Callable

| Runnable | Callable |
|----------|----------|
| `void run()` — no return | `V call()` — returns result |
| Cannot throw checked exceptions | Can throw checked exceptions |
| Java 1.0 | Java 1.5 |
| Used with `Thread` | Used with `ExecutorService` |

```java
// Runnable — no return
Runnable task = () -> System.out.println("Runnable running");
new Thread(task).start();

// Callable — returns Future
Callable<Integer> callable = () -> {
    Thread.sleep(1000);
    return 42;
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(callable);
System.out.println("Result: " + future.get());  // 42 (blocks until done)
executor.shutdown();
```

---

## Q49. ExecutorService

A **thread pool management** framework that decouples task submission from thread management.

```java
// Fixed thread pool — reuses 5 threads
ExecutorService pool = Executors.newFixedThreadPool(5);

// Submit tasks
for (int i = 0; i < 20; i++) {
    final int taskId = i;
    pool.submit(() -> {
        System.out.println("Task " + taskId + " by " + Thread.currentThread().getName());
    });
}

pool.shutdown();             // no new tasks, complete existing
pool.awaitTermination(10, TimeUnit.SECONDS);
```

**Types of Pools:**
| Pool Type | Description |
|-----------|-------------|
| `newFixedThreadPool(n)` | Exactly n threads |
| `newCachedThreadPool()` | Creates threads as needed, reuses idle |
| `newSingleThreadExecutor()` | Only 1 thread — tasks run sequentially |
| `newScheduledThreadPool(n)` | For delayed/periodic tasks |

> "Always use ExecutorService over raw `new Thread()`. It manages lifecycle, prevents resource exhaustion, and supports Future for results."

---

## Q50. Synchronized Block vs Method

```java
// Synchronized METHOD — locks entire method on 'this'
public synchronized void transfer(int amount) {
    balance -= amount;
}

// Synchronized BLOCK — locks only critical section (finer control)
public void transfer(int amount) {
    // non-critical code here (no lock)
    synchronized(this) {
        balance -= amount;  // only this part locked
    }
    // non-critical code here (no lock)
}

// Lock on specific object (best practice)
private final Object lock = new Object();
public void transfer(int amount) {
    synchronized(lock) {
        balance -= amount;
    }
}
```

| Synchronized Method | Synchronized Block |
|---------------------|-------------------|
| Locks entire method | Locks specific code section |
| Lock on `this` (instance) or class | Can lock on any object |
| Less flexible | More flexible & better performance |
| Coarse-grained | Fine-grained |

---

## Q51. What is Race Condition?

A **race condition** occurs when multiple threads access shared data **simultaneously**, and the final result depends on **execution order**.

```java
class Counter {
    private int count = 0;

    // NOT thread-safe!
    void increment() { count++; }  // READ → MODIFY → WRITE (3 steps!)
}

// Thread A reads count=5, Thread B reads count=5
// Both increment to 6 — but expected 7!

// FIX 1: synchronized
synchronized void increment() { count++; }

// FIX 2: AtomicInteger (lock-free, faster)
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();  // atomic operation
```

**Real-world:** Two people booking the LAST train ticket simultaneously — both see 1 ticket available, both book successfully = overbooking!

---

## Q52. Thread Pool

A **pool of pre-created threads** that are reused for executing tasks, avoiding the overhead of creating/destroying threads repeatedly.

```
Task Queue: [T1] [T2] [T3] [T4] [T5] [T6]
                    ↓
Thread Pool:  [Thread-1] [Thread-2] [Thread-3]
              takes T1    takes T2    takes T3
              finishes    finishes    finishes
              takes T4    takes T5    takes T6
```

**Benefits:** Reduced overhead, controlled resource usage, better performance, prevents OutOfMemoryError from unlimited threads.

---

## Q53. Future Object

**Future** represents the result of an asynchronous computation.

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

Future<String> future = executor.submit(() -> {
    Thread.sleep(2000);
    return "Data from database";
});

// Do other work while task runs...
System.out.println("Processing other things...");

// Now get result (blocks if not ready)
if (future.isDone()) {
    String result = future.get();
} else {
    String result = future.get(5, TimeUnit.SECONDS);  // timeout
}

future.cancel(true);  // cancel if still running

executor.shutdown();
```

---

## Q54. Deadlock Prevention Techniques

**Deadlock:** Two or more threads forever waiting for each other's locks.

```java
// DEADLOCK EXAMPLE
Object lockA = new Object(), lockB = new Object();

Thread t1 = new Thread(() -> {
    synchronized(lockA) {
        Thread.sleep(100);
        synchronized(lockB) { /* work */ }  // waiting for lockB
    }
});

Thread t2 = new Thread(() -> {
    synchronized(lockB) {
        Thread.sleep(100);
        synchronized(lockA) { /* work */ }  // waiting for lockA
    }
});
// t1 holds lockA, waits lockB. t2 holds lockB, waits lockA → DEADLOCK!
```

**Prevention Techniques:**
1. **Lock Ordering:** Always acquire locks in same order
```java
// Both threads acquire lockA first, then lockB — no deadlock
synchronized(lockA) { synchronized(lockB) { /* work */ } }
```
2. **tryLock with timeout** (ReentrantLock)
```java
ReentrantLock lock = new ReentrantLock();
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try { /* work */ } finally { lock.unlock(); }
} else { /* handle failure */ }
```
3. **Avoid nested locks**
4. **Use `java.util.concurrent` utilities** (Semaphore, CountDownLatch)

---

## Q55-Q57. start() vs run(), start() called twice, synchronized keyword

**start() vs run():**
| `start()` | `run()` |
|-----------|---------|
| Creates new thread, calls `run()` in it | Runs in current thread (no new thread!) |
| `t.start()` → multithreading | `t.run()` → just a method call |

**start() called twice:** Throws `IllegalThreadStateException`.

**synchronized keyword:**
```java
class SharedPrinter {
    synchronized void print(String doc) {   // only 1 thread at a time
        System.out.print("[" + doc);
        try { Thread.sleep(500); } catch(Exception e) {}
        System.out.println("]");
    }
}
// Without synchronized: [Doc1[Doc2]  ]  — interleaved!
// With synchronized:    [Doc1]  [Doc2]  — orderly
```

---

## Q58-Q59. Real-life multithreading example & Thread Scheduling

**Real-life examples:**
- **Web Server:** Each HTTP request handled by a separate thread from pool
- **Video Player:** One thread plays video, another plays audio, another handles UI
- **Banking App:** Multiple ATM transactions processed simultaneously

**Thread Scheduling:**
JVM uses **preemptive scheduling** — OS allocates CPU time slices. Java provides hints:
- `Thread.setPriority(1-10)` — MIN=1, NORM=5, MAX=10
- `Thread.yield()` — suggests current thread pause (scheduler may ignore)
- `Thread.sleep(ms)` — guaranteed pause
- Actual scheduling depends on OS — Java doesn't guarantee order

---

## Q60-Q63. Deadlock example & prevention (covered in Q54)

**Deadlock conditions (all 4 must hold):**
1. **Mutual Exclusion** — resource held exclusively
2. **Hold and Wait** — holding one, waiting for another
3. **No Preemption** — can't forcibly take a lock
4. **Circular Wait** — A waits for B, B waits for A

**Break any one condition to prevent deadlock.** Lock ordering breaks circular wait.
