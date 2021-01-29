# summer_practice_jb

## NUMA-Aware Scheduler in Kotlin Coroutines

The current coroutines scheduler (aka dispatcher) maintains per-thread queues and performs stealing when the local queue becomes empty. However, the choice of the queue for stealing does not depend on the NUMA architecture. This way, it is possible to access a queue from another NUMA node, which is expensive comparing to stealing from the same-node queue.

In short, you will make the scheduler NUMA-friendly by preferring stealing from the same node queues. The project is technically untrivial since it requires using NUMA-related API, which is inaccessible in JVM by default. Thus, you will need to use JNI (Java Native Interface) to access this API. In addition, you will contribute a set of macro benchmarks to evaluate the performance boost.

## Multi-Word CAS Optimization and Evaluation

The state-of-the-art k-CAS algorithm by Harris et al. [1] uses a special descriptor for the k-CAS operation itself and a special restricted double-compare single-swap (RDCSS) operation to replace the expected value with the k-CAS descriptor atomically if the k-CAS operation is not completed yet. The standard way to implement this RDCSS operation is by using special descriptors so that it requires two additional CAS-s to set this descriptor and make a consensus on its outcome. Recently, there was presented an algorithm that achieves this atomicity ``for free'', without RDCSS, by tricky descriptors reclamation [2]. However, this solution requires a custom epoch-based memory reclamation.

We suggest an optimization that makes the algorithm obstruction-free but uses only one descriptor and does not require specific memory reclamation; it should work great as a fast path for the classic algorithm. We also believe that it is possible to create more optimization and improve the multi-word CAS performance.

In sum, you will implement all existing k-CAS algorithms in C++ (and probably Kotlin), write multiple benchmarks, and create and evaluate new optimizations. You can find an actual list of existing k-CAS implementations in [2], see Table 1 in Section 8.

[1] Harris, Timothy L., Keir Fraser, and Ian A. Pratt. "A practical multi-word compare-and-swap operation." International Symposium on Distributed Computing. Springer, Berlin, Heidelberg, 2002. https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.16.6655&rep=rep1&type=pdf

[2] Guerraoui, R., Kogan, A., Marathe, V. J., & Zablotchi, I. (2020). Efficient multi-word compare and swap. arXiv preprint arXiv:2008.02527. https://arxiv.org/pdf/2008.02527


## Recoverable Concurrent Algorithms for NVRAM

With the rise of non-volatile memory (NVRAM), the corresponding recoverable algorithms -- algorithms that leverage state saved in NVRAM for efficient recovery from crash failures -- became a hot topic in the concurrent programming field. See the corresponding SPTDC talk by Danny Hendler: https://2019.sptdc.ru/2019/talks/2flspfacqcvwd5zykt2ngs/

Recently, we supported several popular models for such algorithms in Lincheck -- the tool for testing concurrency on JVM (https://github.com/Kotlin/kotlinx-lincheck). With such a tool developing new algorithms, the ones for NVRAM, in particular, becomes simpler and more efficient. Thus, we suggest creating new algorithms!

All in all, you will read a lot of papers related to NVRAM, choose some topic(s) (there are a lot of data structures that are still not implemented, so we have different options), and create a new algorithm.

## Academic concurrent algorithms evaluation

Lincheck is a practical tool for testing concurrent algorithms in JVM-based languages, which supports both stress-testing and bounded model checking. Roughly, Lincheck generates a series of concurrent scenarios, executes them in either stress-testing or model-checking mode, and checks whether there exists some sequential execution that can explain the results satisfying the specified correctness property. In addition, it supports durable data structures designed for NVRAM (non-volatile memory), but in a very naive way, assuming that all writes are automatically persisted, and the operations can detect whether they are completed or not after a crash. See the project website for details: https://github.com/Kotlin/kotlinx-lincheck.

Under this project, we suggest implementing a lot of algorithms presented at the recent top-tier academic conferences and checking whether they are correct with Lincheck. We expect to find several bugs either in the provided Java implementations or in the algorithms themselves. The model checking mode should help a lot with the investigations since it provides a trace of the incorrect execution.

Working on this project, you will get a lot of knowledge and practice in concurrent algorithms.
