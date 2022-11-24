# 1. Comparing Fair Readers-Writer Locks
Recently, we introduced a framework for implementing fair synchronization primitives, such as the mutex, the semaphore, the barrier, and the count-down-latch. This framework is called `CancellableQueueSynchronizer` (https://arxiv.org/abs/2111.12682). Additionally to that, we also developed a readers-writer mutex algorithm. It would be interesting to evaluate the resulting performance, comparing this solution with other readers-writer mutexes and optimizing the algorithm. 

During the internship, you will read many papers on readers-writer mutexes, implement the corresponding algorithms, and compare them on real-world benchmarks.  

Strong knowledge of algorithms and Java/Kotlin or C++ is required. Experience in implementing lock-free data structures would be helpful. 

# 2. Comparing Coroutines Schedulers
Popular programming languages, such as Kotlin, Java, or C#, provide a way to write asynchronous code via coroutines or `async/await` construct. To execute coroutines or continuations quickly, we need an efficient scheduling algorithm. With almost every popular language providing its solution, it is nearly impossible to compare different approaches, as the language environments are different. However, we can implement all the schedulers for one language (e.g., for Kotlin) and, thus, compare them under the same environment. In Kotlin, we have officially provided schedulers for Kotlin Coroutines and Java Fibers (aka Project Loom), as well as an implementation of Go's scheduling algorithm. It would be interesting to ådd the C# scheduler implement and compare all of them, figuring out how algorithm differences affect performance under real-world workloads. 

During the internship, you will implement the existing coroutine scheduler in C# for Kotlin Coroutines, develop real-world benchmarks, and compare the scheduling algorithms.

Strong knowledge of algorithms and Java/Kotlin is required.
