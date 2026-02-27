# The Core Hierarchy (From Hardware to Software)

- Core: A physical processing unit in your CPU. Each core can run exactly one instruction stream at a time. If a core supports "Hyper-threading," it acts as two logical cores.
- Process: An instance of a running program (like your JVM). It has its own isolated memory space. One process can contain multiple threads.
- OS Thread: A "native" thread managed by the Operating System kernel. The OS scheduler decides which OS thread gets to run on which physical Core

#### Java Thread Types
- **Plateform Thread:**  A thin wrapper around an OS Thread. It is "heavy" because it requires ~1MB of memory for its stack and is scheduled by the OS.
- **Virtual Thread:**  A thin wrapper around an OS Thread. It is "heavy" because it requires ~1MB of memory for its stack and is scheduled by the OS.

#### How they Connect (The Mounting Process)
JVM acts as a middleman between your code and the hardware;
1.  **Virtual Thread** -> **Carrier(Plateform) Thread:** When a virtual Thread needs to run, the JVM `mounts` it onto a platform Thread (called a `Carrier Thread` in this context)
2.  **Carrier Thread -> OS Thread:** Thre Carrier Thread is a standard platform thread, which is already mapped 1:1 to an OS Thread.
3. **OS Thread -> Core**  The OS schdules that OS thread to run on a physical CPU core.

4. #### . How the JVM Manages Them (The `Magic` of Yielding)
   - The primary advantage of Virtual Threads is how they handle blocking (e.g., waiting for a database or API):
   - Traditional (Platform) Threads: If a Platform Thread waits for I/O, the underlying OS Thread is blocked and sits idle, wasting memory and CPU potential.
   - Virtual Threads: When a Virtual Thread hits a blocking operation, the JVM unmounts it from the Carrier Thread.
     - The Virtual Thread's state is moved to the Heap.
     - The Carrier (Platform) Thread is now free to "mount" a different Virtual Thread and keep the CPU busy.
     - Once the I/O is done, the JVM resumes the original Virtual Thread on any available Carrier Thread.