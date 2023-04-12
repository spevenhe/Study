This is based on
http://huyue.xn--6qq986b3xl/2022/06/20/%E4%BD%A0%E4%B8%8D%E7%9F%A5%E9%81%93%E7%9A%84Nova%E4%B9%8BCPU%E6%8B%93%E6%89%91/

Background knowledge supplemnts:
# 计算平台体系结构

## 1 SMP 
（Sysmmetric Multi-Processor，对称多处理器），顾名思义，SMP 由多个具有对称关系的处理器组成。所谓对称，即处理器之间是水平的镜像关系，无有主从之分。SMP 的出现使一台计算机不再由单个 CPU 组成。

SMP 的典型特征为**「多个处理器共享一个集中式存储器」**，且每个处理器访问存储器的时间片相同，使得工作负载能够均匀的分配到所有可用处理器上

![image](https://user-images.githubusercontent.com/42630862/231361825-3f6e87d7-1aaa-4bc4-98ad-49cb549b98ee.png)
![image](https://user-images.githubusercontent.com/42630862/231365917-fd0c02ca-3037-4dc0-9f39-05074e9d957f.png)


## 2 NUMA 非统一内存访问结构

现代计算机系统中，处理器的处理速度已经超过了主存的读写速度，限制计算机计算性能的瓶颈转移到了存储器带宽之上。SMP 由于集中式共享存储器的设计限制了处理器访问存储器的频次，导致处理器可能会经常处于对数据访问的饥饿状态。

**NUMA（Non-Uniform Memory Access，非一致性存储器访问）**的设计理念是将处理器和存储器划分到不同的节点（NUMA Node），它们都拥有几乎相等的资源。在 NUMA 节点内部会通过自己的存储总线访问内部的本地内存，而所有 NUMA 节点都可以通过主板上的共享总线来访问其他节点的远程内存。

![image](https://user-images.githubusercontent.com/42630862/231362241-03ab17c3-6182-4de8-becc-0a5a49c0d81b.png)

## 3 SMT
SMT架构
SMT（Simultaneous Multithreading，同步多线程）是一种将硬件多线程与超标量处理器技术相结合的处理器设计。SMT把CPU的每个物理内核拆分为虚拟内核，这些虚拟内核被称为线程，这样做是为了提高性能并允许每个内核一次运行两个指令流。

SMT是SMP架构的设计补充。SMP架构中的CPU共享一条总线和存储，而SMT架构中CPU共享更多的组件。共享组件的CPU被称为Siblings。所有的CPU在系统上都显示为可用CPU，并且可以执行工作负载。但是，与NUMA一样，都会有线程竞争共享资源的情况。

Intel Hyper-Threading Technolog（超线程技术）和SMT完全一样，都是允许在每个内核上运行多个线程。

## 4 MPP 
MPP（Massive Parallel Processing，大规模并行处理），既然 NUMA 扩展性的限制是没有完全实现资源（e.g. 存储器、互联模块）的隔离性，那么 MPP 的解决思路就是为处理器提供彻底的独立资源。

MPP 拥有多个真正意义上的独立的 SMP 单元，每个 SMP 单元独占并只会访问自己本地的内存、I/O 等资源，SMP 单元间通过节点互联网络进行连接（Data Redistribution，数据重分配），是一个完全无共享（Share Nothing）的 CPU 计算平台结构。
![image](https://user-images.githubusercontent.com/42630862/231362764-bb6592dc-c0f6-43be-976c-cdc1e54e8d41.png)


# numa 概念
Node：包含有若干个 Socket 的组

Socket：表示一颗物理 CPU 的封装，简称插槽。Intel 为了避免将物理处理器和逻辑处理器混淆，所以将物理处理器统称为插槽。

Core：Socket 内含有的物理核。

Thread：在具有 Intel 超线程技术的处理器上，每个 Core 可以被虚拟为若干个（通常为两个）逻辑处理器，逻辑处理器会共享大多数内核资源（e.g. 内存缓存、功能单元）。逻辑处理器被统称为 Thread。

Processor：处理器的统称，可以区分为物理处理器（physical processor）和逻辑处理器（virtual processors），对于大多数应用程序而言，它们并不关心处理器是物理的还是逻辑的。

Siblings：表示每一个 physical processor 所含有的 virtual processors 的数量。（If the number of siblings is equal to the number of cores then you have CPUs which are not hyperthreading or hyperthreading is switched off, If the number of siblings is 2x the number of cores then you have a hyperthreading CPU with hyperthreading switched on. ）

![image](https://user-images.githubusercontent.com/42630862/231363226-cada259a-2401-48bb-9e69-f94a3f92043f.png)
EXAMPLE：上图为一个 NUMA Topology，表示该服务器具有 2 个 numa node，每个 node 含有一个 socket，每个 socket 含有 6 个 core，每个 core 又被超线程为 2 个 thread，所以服务器总共的 processor = 2*1*6*2 = 24 颗。

# NUMA 调度策略

Linux 的每个进程或线程都会延续父进程的 NUMA 策略，优先会将其约束在一个 NUMA node 内。当然了，如果 NUMA 策略允许的话，进程也可以调用其他 node 上的资源。

NUMA 的 CPU 分配策略有下列两种：

cpunodebind：约束进程运行在指定的若干个 node 内

physcpubind：约束进程运行在指定的若干个物理 CPU 上

NUMA 的 Memory 分配策略有下列 4 种：

localalloc：约束进程只能请求访问本地内存

preferred：宽松地为进程指定一个优先 node，如果优先 node 上没有足够的内存资源，那么进程允许尝试运行在别的 node 内

membind：规定进程只能从指定的若干个 node 上请求访问内存，并不严格规定只能访问本地内存

interleave：规定进程可以使用 RR 算法轮转地从指定的若干个 node 上请求访问内存

因为NUMA默认的内存分配策略是localalloc，优先在进程所在CPU的本地内存中分配，会导致CPU节点之间内存分配不均衡，当某个CPU节点的内存不足时，会导致Swap产生，而不是从远程节点分配内存。这就是所谓的Swap Insanity现象。


