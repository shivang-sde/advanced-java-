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
> **Important** Never Call the `run()` meghod directly 