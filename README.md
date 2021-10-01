# 1. Adopting GenMC Model Checker to Lincheck
Concurrent programming is notoriously complex and error-prone. Programming bugs can arise from various sources, such as operation re-reordering or an incomplete understanding of the memory model. With weaker-memory consistency architectures, such as ARM on the rise due to Apple efforts, ensuring that your concurrent algorithms are correct is as important as ever.

Some time ago, we presented Lincheck — a new practical tool for testing concurrent algorithms in JVM-based languages, such as Java, Kotlin, or Scala, that supports both stress testing and bounded model checking modes ([GitHub](https://github.com/Kotlin/kotlinx-lincheck)). With such a tool, developing new algorithms becomes simpler and significantly more efficient. Please, read [this blog post](https://blog.jetbrains.com/kotlin/2021/02/how-we-test-concurrent-primitives-in-kotlin-coroutines) before going to the rest of the project description.

Since the main implementation language of Lincheck is Kotlin, which is multi-platform with interoperability with native languages like Swift or C/C++, one of our current main focuses is making it possible to write Lincheck tests for implementations in these languages. Currently, we have successfully supported the stress testing mode in Kotlin/Native, which automatically makes it possible to use Lincheck in C/C++.

With a possibility to write tests in C/C++, it becomes intriguing to adopt the existing state-of-the-art model checkers to Lincheck. As mentioned in the beginning, it is vital to support weak memory models. Therefore, we consider two main model checkers: Nidhugg ([GitHub](https://github.com/nidhugg/nidhugg)) and GenMC ([website](https://plv.mpi-sws.org/genmc/), [GitHub](https://github.com/mpi-sws/genmc/)), both work on the level of LLVM bitcode.

Under this project, you will you integrate GenMC into Lincheck.


# 2. Concurrent Cache-Friendly Splay Tree
Splay trees are known as contention-adjustable balanced trees, which work efficiently under workloads with non-uniform access. However, they are expensive in terms of cache locality (each element is associated with a node) and are not easily portable to the concurrent environment. There are several ideas on how to fill the gap. First, each node can contain multiple elements, while the number of children remains two. After that, we can increase the number of children up to `K+1`, where `K` is the number of elements stored in each node. (It is interesting, that we can maintain `K/2` children, what potentially decreases the balancing cost.) 
Also, it is possible to apply the move-to-front heuristic under some probability and/or perform it in batches. In the end, we can design a concurrent solution with wait-free reads and lock/obstruction-free updates, performing balancing lazily; thus, solving potential races and reducing contention. This project requires strong algorithmic knowledge. 

# 3. Priority Scheduler for Kotlin Coroutines
Recently, we proposed SMQ priority scheduler (the draft is available [here](https://arxiv.org/abs/2109.00657))-- the Stealing Multi-Queue algorithm which leverages the classic Multi-Queue design ([paper](https://dl.acm.org/doi/pdf/10.1145/2755573.2755616?casa_token=qJjo7gi9wDAAAAAA:GY-a0HzShpGKj0cz9l7yHQw5z56YMNXaxR4DBRenTFCRzGPS4Stxx8X9py-dB7_a5gvXZcBhxXoS)) to reduce the amount of wasted work in iterative algorithms ([paper](https://dl.acm.org/doi/pdf/10.1145/3323165.3323201?casa_token=tkv3jP8IUb4AAAAA:rvEiqmZLcneERZu4po_3ZUcyhrgfDv1761vNoI0zcM9PyWX-hiWf6H7yTFZMvTOkpdSgSif6yF-V)). (Essentially, priority schedulers are concurrent priority queues with relaxed semantics.) We successfully showed that the presented approach works great for parallel graph algorithms, outperforming the state-of-the-art solutions. Moreover, it is possible to use the same design in other applications, such as coroutines scheduling, when each coroutine can have a priority. However, it brings us to a set of problems.

First, the classic SMQ approach does not utilize threads efficiently; all of them consume CPU until the total work is done, and we need to make it possible for threads to sleep and wake up only when required. What is more, this feature can improve even the existing graph processing algorithms when the best performance is obtained not at the maximal number of threads.

The next step is to make the algorithm fully adaptive -- the current version has a couple of parameters to be tuned. This tuning should be performed automatically, depending on the current workload, and the overhead for this adaptivity should be minimum. In some sense, the number of active threads is also such an adaptive property.

To evaluate, we will implement the algorithm in C++ and benchmark it against the state-of-the-art priority schedulers.

The last step is to integrate the algorithm into Kotlin Coroutines as a separate library and provide a set of examples on how Kotlin Coroutines can be used for iterative algorithms with this coroutines scheduler under the hood.

# 4. Comparing Coroutine Schedulers from Different Languages
Kotlin has coroutines, which can be considered as tasks that execute on threads. To execute coroutines efficiently, we need a special scheduling algorithm. Surprisingly, almost every popular language is equipped with its own algorithm, and it is almost impossible to compare the solutions as the language environments are different. However, we can implement all the schedulers for one language (for Kotlin, obviously), and, thus, compare them under the same environment. In Kotlin, we already have schedulers for Kotlin Coroutines and Java Fibers (aka Project Loom); the last one is the classic ForkJoinPool in the asynchronous mode from the standard Java library. Also, it would be interesting to consider Go and C# schedulers and implement them for Kotlin Coroutines. After that, we need to create a set of benchrmarks to compare all the schedulers under different (mostly real-world) workloads. 



# 5. Practical Multi-Word CAS: Optimizations and Evaluation
The k-CAS operation (also known as CASN) is a widely-used abstraction that allows to atomically replace the expected values with new ones in the specified k memory locations. It succeeds only when the current values coincide with the expected ones for all locations. Thus, k-CAS with k = 1 is the standard CAS operation which manipulates with a single memory location. This project is aimed at making the k-CAS implementation as efficient as possible for various practical applications.

The state-of-the-art k-CAS algorithm by Harris et al. [1] uses a special descriptor for the k-CAS operation itself (this descriptor represents the operation and manages its completion status) and a special restricted double-compare single-swap (RDCSS) operation to replace the expected value with the k-CAS descriptor atomically if the k-CAS operation is not completed yet. (Please refer to the original paper for the problem details.) The standard way to implement this RDCSS operation is to use special descriptors, so the operation requires two additional CAS-s to set this additional descriptor and make a consensus on its outcome. Recently, there was presented an algorithm that achieves this atomicity "for free," without RDCSS, by tricky descriptors reclamation [2]. However, this solution requires a custom epoch-based memory reclamation.

We suggest an optimization that makes the algorithm obstruction-free but utilizes only one descriptor and does not require specific memory reclamation; it should work great as a fast path for the classic algorithm. We also believe that it is possible to create more optimization by relaxing the progress guarantee and improve the multi-word CAS performance in practical scenarios.

In sum, you will implement all existing k-CAS algorithms in C++ (and, probably, Kotlin), write multiple real-world benchmarks, and create and evaluate new optimizations. You can find an actual list of existing k-CAS implementations in [2]; see Table 1 in Section 8 in the paper.

[1] Harris, Timothy L., Keir Fraser, and Ian A. Pratt. "A practical multi-word compare-and-swap operation." International Symposium on Distributed Computing. Springer, Berlin, Heidelberg, 2002.
https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.16.6655&rep=rep1&type=pdf

[2] Guerraoui, R., Kogan, A., Marathe, V. J., & Zablotchi, I. (2020). Efficient multi-word compare and swap. arXiv preprint arXiv:2008.02527.
https://arxiv.org/pdf/2008.02527
