# 参考文献

> [bibliography.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/bibliography.md)
> <br>
> commit c158fd93b81f9dd5b4f28723f256760b21eab44f

这是一个与 Rust 相关的材料的阅读列表。这包含了曾经影响过 Rust 先验研究，以及关于 Rust 的出版物。

> **（注：以下翻译属个人理解，勿作为参考！）**

## 类型系统

* [Cyclone语言中基于区域的内存管理（Region based memory management in Cyclone）](http://209.68.42.137/ucsd-pages/Courses/cse227.w03/handouts/cyclone-regions.pdf)
* [Cyclone语言中的手动安全内存管理（Safe manual memory management in Cyclone）](http://www.cs.umd.edu/projects/PL/cyclone/scp.pdf)
* [类型类：使临时多态不再临时（Typeclasses: making ad-hoc polymorphism less ad hoc）](http://www.ps.uni-sb.de/courses/typen-ws99/class.ps.gz)
* [宏综述（Macros that work together）](https://www.cs.utah.edu/plt/publications/jfp12-draft-fcdf.pdf)
* [特性：组合类型的行为（Traits: composable units of behavior）](http://scg.unibe.ch/archive/papers/Scha03aTraits.pdf)
* [消除别名（Alias burying）](http://www.cs.uwm.edu/faculty/boyland/papers/unique-preprint.ps) - 我们尝试了一些相似的内容并放弃了它
* [外部唯一性是足够的（External uniqueness is unique enough）](http://www.computingscience.nl/research/techreps/repo/CS-2002/2002-048.pdf)
* [用于安全并行的唯一性和引用不可变性（Uniqueness and Reference Immutability for Safe Parallelism）](https://research.microsoft.com/pubs/170528/msr-tr-2012-79.pdf)
* [基于区域的内存管理（Region Based Memory Management）](http://www.cs.ucla.edu/~palsberg/tba/papers/tofte-talpin-iandc97.pdf)

## 并发

* [Singularity：软件栈的重新思考（Singularity: rethinking the software stack）](https://research.microsoft.com/pubs/69431/osr2007_rethinkingsoftwarestack.pdf)
* [Singularity操作系统中支持快速和可靠的消息传递的语言（Language support for fast and reliable message passing in singularity OS）](https://research.microsoft.com/pubs/67482/singsharp.pdf)
* [通过work stealing来安排多线程计算（Scheduling multithreaded computations by work stealing）](http://supertech.csail.mit.edu/papers/steal.pdf)
* [多道程序多处理器的线程调度（Thread scheduling for multiprogramming multiprocessors）](http://www.eecis.udel.edu/~cavazos/cisc879-spring2008/papers/arora98thread.pdf)
* [work stealing中的数据局部性（The data locality of work stealing）](http://www.aladdin.cs.cmu.edu/papers/pdfs/y2000/locality_spaa00.pdf)
* [动态环形work stealing双端队列（Dynamic circular work stealing deque）](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.170.1097&rep=rep1&type=pdf) - Chase/Lev双端队列
* [异步-完成并行的work优先和help优先的调度策略（Work-first and help-first scheduling policies for async-finish task parallelism）](http://www.cs.rice.edu/~yguo/pubs/PID824943.pdf) - 比严格的work stealing更宽泛
* [一个Java的fork/join灾难（A Java fork/join calamity）](http://www.coopsoft.com/ar/CalamityArticle.html) - 对Java fork/join库的批判，特别是其在非严格计算时的work stealing实现
* [并发系统的调度技巧（Scheduling techniques for concurrent systems）](http://www.ece.rutgers.edu/~parashar/Classes/ece572-papers/05/ps-ousterhout.pdf)
* [竞争启发调度（Contention aware scheduling）](http://www.blagodurov.net/files/a8-blagodurov.pdf)
* [时间共享多核系统的平衡work stealing（Balanced work stealing for time-sharing multicores）](http://www.cse.ohio-state.edu/hpcs/WWW/HTML/publications/papers/TR-12-1.pdf)
* [三层蛋糕？（Three layer cake）](http://www.upcrc.illinois.edu/workshops/paraplop10/papers/paraplop10_submission_8.pdf)
* [非阻塞半work stealing队列（Non-blocking steal-half work queues）](http://www.cs.bgu.ac.il/~hendlerd/papers/p280-hendler.pdf)
* [Reagents：表现和编写细粒度的并发（Reagents: expressing and composing fine-grained concurrency）](http://www.mpi-sws.org/~turon/reagents.pdf)
* [用于共享内存多处理器的可扩展同步性的算法（Algorithms for scalable synchronization of shared-memory multiprocessors）](https://www.cs.rochester.edu/u/scott/papers/1991_TOCS_synch.pdf)
* [Epoch-based reclamation](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-579.pdf).

## 其它

* [只能崩溃的软件（Crash-only software）](https://www.usenix.org/legacy/events/hotos03/tech/full_papers/candea/candea.pdf)
* [编写高性能内存分配器（Composing High-Performance Memory Allocators）](http://people.cs.umass.edu/~emery/pubs/berger-pldi2001.pdf)
* [对手动内存分配的思考（Reconsidering Custom Memory Allocation）](http://people.cs.umass.edu/~emery/pubs/berger-oopsla2002.pdf)

## **关于** Rust 的论文

* [Rust中的GPU编程（GPU programming in Rust）](http://www.cs.indiana.edu/~eholk/papers/hips2013.pdf)
* [并行闭包：一个基于老观点的新做法（Parallel closures: a new twist on an old idea）](https://www.usenix.org/conference/hotpar12/parallel-closures-new-twist-old-idea) - 并不完全关于Rust，不过是Nicholas D. Matsakis写的
* [Patina: A Formalization of the Rust Programming Language](ftp://ftp.cs.washington.edu/tr/2015/03/UW-CSE-15-03-02.pdf)。一类型系统子集的早期形式，Eric Reed著。
* [Experience Report: Developing the Servo Web Browser Engine using Rust](http://arxiv.org/abs/1505.07383)。Lars Bergstrom著。
* [Implementing a Generic Radix Trie in Rust](https://michaelsproul.github.io/rust_radix_paper/rust-radix-sproul.pdf)。Michael Sproul的毕业论文。
* [Reenix: Implementing a Unix-Like Operating System in Rust](http://scialex.github.io/reenix.pdf)。Alex Light的毕业论文。
* [Evaluation of performance and productivity metrics of potential programming languages in the HPC environment](http://doc.rust-lang.org/stable/book/academic-research.html)。Florian Wilkens的学士学位论文。比较C，Go和Rust。
* [Nom, a byte oriented, streaming, zero copy, parser combinators library in Rust](http://spw15.langsec.org/papers/couprie-nom.pdf)。Geoffroy Couprie著，关于VLC的研究。
* [Graph-Based Higher-Order Intermediate Representation](http://compilers.cs.uni-saarland.de/papers/lkh15_cgo.pdf)。一个用Impala（一个类似Rust的语言）实现的实验性的IR。
* [Code Refinement of Stencil Codes](http://compilers.cs.uni-saarland.de/papers/ppl14_web.pdf)。另一个使用Impala的论文。
* [Parallelization in Rust with fork-join and
  friends](http://publications.lib.chalmers.se/records/fulltext/219016/219016.pdf). Linus
  Farnstrand's master's thesis.
* [Session Types for
  Rust](http://munksgaard.me/papers/laumann-munksgaard-larsen.pdf). Philip
  Munksgaard's master's thesis. Research for Servo.
* [Ownership is Theft: Experiences Building an Embedded OS in Rust - Amit Levy, et. al.](http://amitlevy.com/papers/tock-plos2015.pdf)
* [You can't spell trust without Rust](https://raw.githubusercontent.com/Gankro/thesis/master/thesis.pdf). Alexis Beingessner's master's thesis.
