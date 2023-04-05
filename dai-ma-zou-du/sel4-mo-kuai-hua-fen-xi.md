# seL4 模块化分析

#### 模块划分

这里主要对seL4内核进行划分。

* Boot：内核的入口，负责初始化内存和 untyped，填充 `NDKS_BOOT` 这个数据结构。创建 `Idle Thread`。初始化中断向量表。
* Root Server：创建和初始化 `Root Server` 的内核对象，，创建各种子对象对应的cap（CSpace）， 创建 Vspace， 创建和初始化Root Server的虚拟内存映射。初始化 `Root Server` 的 `Initial Thread` 。
* CSpace：提供各种Cap和CNode、TCBCNode的定义以及定义在其上的操作接口。这个是最独立的，不依赖其他任何模块。
* VSpace：首先在 boot 阶段提供对内核地址空间的的VSpace的初始化。其次还提供各种map操作和查找操作。
* Schedule：全局调度的相关数据和对应的调度接口。
* Trap：定义用户态和内核态的切换接口。
* Object：定义各种内核对象，以及在上面可以完成的一些操作接口。
  * untyped
  * TCB
  * endpoint
  * notification
* Syscall：主要用于处理中断、异常和系统调用。这里的依赖比较复杂。几乎依赖了上面提到的所有模块。

<figure><img src="../.gitbook/assets/seL4_kernel_module.png" alt=""><figcaption></figcaption></figure>

#### 独立性分析

从图中看出，独立性最强的是 `CSpace` 模块。可以做到完全不依赖其他模块，其他模块会更多的依赖它。它提供了各种内核对象的句柄的定义和操作，以及提供了句柄派生等操作接口。

其次是 Object 模块，定义了各种内核对象及可以在上面进行的操作。它是十分依赖于 `CSpace` 模块的。

这两个模块都可以用Rust重写，以静态库的形式提供给 seL4。但是这里有两个问题：

* 原始的seL4 kernel 的编译是将所有的代码拷贝到一个文件下进行编译，模块化之后对代码的架构和构建会有很大的调整。
* 上述的模块划分更多的是功能性模块而非过程性模块，基本无法对某个单独的模块实施异步化。
