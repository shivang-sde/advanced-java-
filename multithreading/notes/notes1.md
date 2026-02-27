# Multithreading?

## 1. Multithreading?

- Modern CPUs have multiple core, multithreading allows a java program to utilize these cores by performing multiple tasks concurrently. This leads to better resources utilization, improved throughput, and more responsive applications (e.g. a ui thread remains responsive while bg work is done). However, with great power comes great responsibilty: threads introduce complexity like race conditions, deadlocks, and memory visibilty issues. we'll tackle those later

## 2. Threads vs. Processes

- Processes: An independent program runing in its own memory space. Processes are heavyweight and communicatw via IPC.
- Thread: A lightweight sub-process, the smallest unit of execution within a process, Threads share the same memory space (heap) and resources, making communication easier but also prone to intergernce.

IN java, the `Thread` class and the `Runnable` interface are the primary ways to create and manage threads.

## 3. Creating Threads

- There is 2 ways to define what a thread should execute. Both ultimately use an instance of `Thread` to run code.

## 3.1. Extending the Thread Class

```java
class MyThread extends Thread {
    @overide
    public void run() {
        sout("Thread running: " + Thread.currentThread().getName());
    }
}
//Usages
MyThrea t1 = new MyThread();
t1.start(); // starts a new thread that excutes run()
```

- **Pros:**
  Simple, especailly for small tasks, Can access `Thread` methods directly (e.g., `getName()`, `setName`).
- **Cons**
  Java doesn't support multiple inheritence, so you can extend any other class. Tight coupling: the task is tied to thread object.

## 3.2 Implementing the Runnable interface

```java 
class MyTask implements Runnable {
    @overide 
    public void run() {
        sout("Task running in thread " + Thread.currentThread().getName());

    }
}
// usages
MyTask task = new Mytask();
Thread t2 = new Thread(task);
t2.start()
```

- **Pros:**
    - Separates task logic from thread management - the same `Ruunable` can be passed to different threads or thread pools. 
    - You can extended another class if needed.
    -  Follow the Strategy design pattern; more flexible and resuable.
  
- **Cons:**
  - Slightly more verbose, but with lambdas(Java 8+) it becomes concise;
  - ```java
    Thread t3 = new Thread(() -> System.out.println("Lambda task"));
t3.start();
    ```

**Interview Question:** *Why is implementing Runnable preferred over extending Thread?*

> **Answer :**  Bcoz Java supports single inheritence; implementing  `Runnable` leaves your class free to extend another class. it also decouples the task from the execution mechanism, promoting better design and reuse.
> **Important** Never Call the `run()` method directly - it will execute in the current thread, not create a new one. ALways call `start()`, which launches a new thread and then invokes `run()` on that thread.

4. ## Thread Lifecycle
   - A thread can be in one of several states at any given time. Understanding these states helps in debuging and desiging correct concurrent applications, The States are defined in `Thread.State` enum.
  
  | State         | Description                                                                 |
|---------------|-----------------------------------------------------------------------------|
| NEW           | A thread that has been created but not yet started.                         |
| RUNNABLE      | The thread is executing in the JVM. It may be waiting for CPU time (ready to run) or actually running. |
| BLOCKED       | The thread is waiting to acquire a monitor lock (synchronized block) that another thread holds. |
| WAITING       | The thread is waiting indefinitely for another thread to perform a particular action (e.g., `Object.wait()`, `Thread.join()` without timeout). |
| TIMED_WAITING | The thread is waiting for a specified time (e.g., `Thread.sleep(time)`, `Object.wait(timeout)`, `Thread.join(millis)`). |
| TERMINATED    | The thread has completed execution (either `run()` returned normally or an exception was thrown). |

```java
NEW → start() → RUNNABLE → (scheduler picks) → Running (still RUNNABLE)
Running → yield() → still RUNNABLE
Running → sleep() → TIMED_WAITING → after time → RUNNABLE
Running → wait() → WAITING → notify() → RUNNABLE (or BLOCKED if lock contention)
Running → try to acquire lock, lock held → BLOCKED → lock acquired → RUNNABLE
Running → join() → WAITING (or TIMED_WAITING) → target thread terminates → RUNNABLE
Running → run() ends → TERMINATED
```
> **Note** A  thread in `RUNNABLE` state may not be actively using the CPU; it could be wiating in the ready queue. The JVM doesn't distinguish between "ready" and "running" - they are both considered RUNNABLE 

5. #### Key methods for Thread Control
   cuase the currently executing thread to pause for the specified number of miliseccond. it is a static method, so it always affects the current thread.

5.1. `Thread.sleep(long mills)`
```java
try{
    System.out.println("Sleeping......")
    Thread.sleep(2000) // pause for 2 seconds 
    System.out.printLn("Awake!");

} catch(InteruptedException e) {
    // this exeception is thrown if another thread interrupts this thread while sleeping.
    // Good practice: restore the interrupt status or handle approprately. 
    Thread.currentThread().interrupt(); //re-interrupt
}
```
> **Important :** `sleep` does not release any locks held by the thread. So if a thread goes to sleep inside a `synchronized` block, other threads cannot enter that block.
> Always handle `InteruptedException` - it's a checked exception.

5.2. `Thread.yield()`
- A hint to the thread scheduler that the current thread is willing to yield its current use of the CPU, The scheduler is free to ignore this hint.
```Thread.yeild();```
- Rarely used in producntion code. it might be useful in spin-wait loops or testing, but modern concurrency utilites are preferred. 
- Does not release locks.
- When you want to give other threads a chance to execute.
- Useful in demos or testing concurrency behavior, but rarely needed in production.
  
5.3 `Thread.join()`
  Allows one thread to wait for the completion of another.
  ```java
  Thread t = new Thread(() -> {
    // do some work
  })
  t.start();
  //Main thread continues...
  //But we need the result from t. so we wait.
  try {
    t.join(); // main thread waits indefinitelty for t to die
  } catch (InterruptedException e){
    Thread.currentThread().interrupt();
  }
  ```
**Variants:**
- join(long millis) – waits at most millis milliseconds.
- join(long millis, int nanos) – fine‑grained timeout.

**How it wokrs internally:** When you call `join()` on a thread, the current thread waits on that thread's monitor (the thread object itself) until the target thread terminates. Termination triggers a notifyALl() on the thread object, waking the waiting thread. (This is why you should not use wait/notify on Thread instances yourself - it could interfere.) 
