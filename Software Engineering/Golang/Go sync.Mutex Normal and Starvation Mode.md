#software_engineering #golang 

### References
https://victoriametrics.com/blog/go-sync-mutex/

### Summary of Mutex in Go: Technical Explanation

#### **Understanding Mutex (MUTual EXclusion)**
A **mutex** in Go is a synchronization primitive that ensures only one goroutine can access shared resources (like variables, maps, or channels) at a time, preventing race conditions.

#### **Why Mutex is Necessary**
- **Race conditions** arise when multiple goroutines simultaneously read and write shared data (e.g., modifying a shared counter), leading to unpredictable results.
- A **race condition** happens because operations like `counter++` aren’t atomic, meaning they consist of multiple steps: read, modify, and write. These steps can be interrupted by other goroutines, resulting in lost updates.

#### **How Mutex Works**
A mutex has two essential operations:
- **Lock**: When a goroutine locks a mutex, it claims the resource exclusively.
- **Unlock**: Once done, the goroutine releases the lock, allowing others to access the resource.

Mutexes ensure that only one goroutine at a time can modify shared data by wrapping critical sections of code (e.g., `counter++`) between `Lock()` and `Unlock()` calls.

#### **Mutex Structure**
In Go, the `sync.Mutex` structure contains two key fields:
- **state (int32)**: Tracks the mutex's state, which encodes:
  - **Locked**: Whether the mutex is currently locked (bit 0).
  - **Woken**: Indicates if a goroutine has been woken up to acquire the mutex (bit 1).
  - **Starvation**: Shows if the mutex is in starvation mode (bit 2).
  - **Waiter**: Tracks how many goroutines are waiting for the mutex (bit 3-31).
- **sema (uint32)**: A semaphore used to signal waiting goroutines when the mutex becomes available.

#### **Mutex Lock Flow**
The `Lock()` method has two paths:
1. **Fast path**: When the mutex is unlocked, a goroutine can quickly acquire it using a **Compare-And-Swap (CAS)** operation, setting the state to `locked`. This path is optimized through inlining, making it highly efficient.
   
2. **Slow path**: If the mutex is already locked, the goroutine enters the slow path, involving:
   - **Spinning**: The goroutine waits in a busy loop, periodically checking if the mutex is free. Spinning helps avoid the overhead of putting the goroutine to sleep if the lock might become available soon.
   - If spinning fails (after 120 cycles on ARM64), the goroutine is added to the **wait queue**, goes to sleep, and waits for a wake-up signal.

#### **Normal Mode vs. Starvation Mode**
- **Normal Mode**: Goroutines are queued in a **first-in, first-out (FIFO)** order. However, new goroutines can compete for the mutex against those in the queue, as they’re already running and can quickly attempt to acquire the lock.
- **Starvation Mode**: If a goroutine waits for more than **1 millisecond** without acquiring the lock, the mutex switches to starvation mode. In this mode:
  - The mutex hands control directly to the front of the queue, ensuring fairness.
  - No new goroutines can compete for the lock until the queue is emptied.

#### **Mutex Unlock Flow**
The `Unlock()` function also has fast and slow paths:
1. **Fast path**: The first bit (locked bit) of the state is cleared, and if no other flags (like waiting goroutines) are set, the mutex becomes fully available.
   
2. **Slow path**: If the state indicates waiting goroutines or the mutex is in starvation mode:
   - **Normal Mode**: The mutex tries to decrement the waiter count and wake up one goroutine by releasing the semaphore.
   - **Starvation Mode**: The lock is handed off directly to the front of the queue using the `runtime_Semrelease()` function, ensuring the waiting goroutine acquires the lock without competition.

#### **Spinning: Why and How**
- Spinning occurs in the slow path to avoid the overhead of putting the goroutine to sleep if the lock might soon be available.
- Spinning is implemented using low-level assembly instructions like `YIELD` and a counter (30 cycles in the example) to control how long the goroutine spins before sleeping.

#### **Conclusion**
Go's `sync.Mutex` is an efficient locking mechanism designed to handle both common and edge cases. Its fast path is optimized for performance, while the slow path manages contention through spinning and queueing mechanisms. Starvation mode ensures fairness for long-waiting goroutines, preventing indefinite blocking.