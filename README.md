# Adopting GenMC Model Checker to Lincheck (industrial)
Concurrent programming is notoriously complex and error-prone. Programming bugs can arise from various sources, such as operation re-reordering or an incomplete understanding of the memory model. With weaker-memory consistency architectures, such as ARM on the rise due to Apple efforts, ensuring that your concurrent algorithms are correct is as important as ever.

Some time ago, we presented Lincheck — a new practical tool for testing concurrent algorithms in JVM-based languages, such as Java, Kotlin, or Scala, that supports both stress testing and bounded model checking modes ([GitHub](https://github.com/Kotlin/kotlinx-lincheck)). With such a tool, developing new algorithms becomes simpler and significantly more efficient. Please, read [this blog post](https://blog.jetbrains.com/kotlin/2021/02/how-we-test-concurrent-primitives-in-kotlin-coroutines) before going to the rest of the project description.

Since the main implementation language of Lincheck is Kotlin, which is multi-platform with interoperability with native languages like Swift or C/C++, one of our current main focuses is making it possible to write Lincheck tests for implementations in these languages. Currently, we have successfully supported the stress testing mode in Kotlin/Native, which automatically makes it possible to use Lincheck in C/C++.

With a possibility to write tests in C/C++, it becomes intriguing to adopt the existing state-of-the-art model checkers to Lincheck. As mentioned in the beginning, it is vital to support weak memory models. Therefore, we consider two main model checkers: Nidhugg ([GitHub](https://github.com/nidhugg/nidhugg)) and GenMC ([website](https://plv.mpi-sws.org/genmc/), [GitHub](https://github.com/mpi-sws/genmc/)), both work on the level of LLVM bitcode.

Under this project, you will you integrate GenMC into Lincheck.


# Concurrent Cache-Friendly Splay Tree
Splay trees are known as contention-adjustable balanced trees, which work efficiently under non-uniform workloads. However, they are expensive in terms of cache locality (each element is associated with a node) and are not easily portable to the concurrent environment. There are several ideas on how to fill the gap. First, each node can contain multiple elements, while the number of children remains two. After that, we can increase the number of children up to `K+1`, where `K` is the number of elements stored in each node. Also, it is possible to apply the move-to-front heuristic under some probability and/or perform it in batches. In the end, we can design a concurrent solution with wait-free reads and lock/obstruction-free updates, performing balancing lazily; thus, solving potential races and reducing contention. This project requires strong algorithmic knowledge. 

# Priority Scheduler for Kotlin Coroutines 
Recently, we proposed SMQ priority scheduler (the draft is available [here](https://arxiv.org/abs/2109.00657))-- the Stealing Multi-Queue algorithm which leverages the classic Multi-Queue design ([paper](https://dl.acm.org/doi/pdf/10.1145/2755573.2755616?casa_token=qJjo7gi9wDAAAAAA:GY-a0HzShpGKj0cz9l7yHQw5z56YMNXaxR4DBRenTFCRzGPS4Stxx8X9py-dB7_a5gvXZcBhxXoS)) to reduce the amount of wasted work in iterative algorithms ([paper](https://dl.acm.org/doi/pdf/10.1145/3323165.3323201?casa_token=tkv3jP8IUb4AAAAA:rvEiqmZLcneERZu4po_3ZUcyhrgfDv1761vNoI0zcM9PyWX-hiWf6H7yTFZMvTOkpdSgSif6yF-V)). (Essentially, priority schedulers are concurrent priority queues with relaxed semantics.) We successfully showed that the presented approach works great for parallel graph algorithms, outperforming the state-of-the-art solutions. Moreover, it is possible to use the same design in other applications, such as coroutines scheduling, when each coroutine can have a priority. However, it brings us to a set of problems.

First, the classic SMQ approach does not utilize threads efficiently; all of them consume CPU until the total work is done, and we need to make it possible for threads to sleep and wake up only when required. What is more, this feature can improve even the existing graph processing algorithms when the best performance is obtained not at the maximal number of threads.

The next step is to make the algorithm fully adaptive -- the current version has a couple of parameters to be tuned. This tuning should be performed automatically, depending on the current workload, and the overhead for this adaptivity should be minimum. In some sense, the number of active threads is also such an adaptive property.

To evaluate, we will implement the algorithm in C++ and benchmark it against the state-of-the-art priority schedulers.

The last step is to integrate the algorithm into Kotlin Coroutines as a separate library and provide a set of examples on how Kotlin Coroutines can be used for iterative algorithms with this coroutines scheduler under the hood.
