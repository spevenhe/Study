https://aijishu.com/a/1060000000388928
https://cloud.tencent.com/developer/article/2353832?areaId=106001

# intel icelake 微架构
![image](https://github.com/spevenhe/Study/assets/42630862/9e2acaca-c5e1-4da4-9e8e-3cf1527e844d)


我们可以简单的将这个微架构图划分为两个部分：Frontend（前端）与Backend（后端）。前端负责从内存中取出指令，并将他们转换成uOps（微指令）。在CISC指令集中，因为指令的复杂性，可能会导致硬件执行关键路径过长而影响性能，所以会将指令先翻译成微指令，再去执行。后端则负责调度前端生成的微指令并执行，最后提交（Commit）这些微指令，让他们退休(Retired)。为了平衡前端和后端，两者之间存在这一个微指令队列，前端生成的微指令会经由这里被送到后端。

**fronted 与 backend 是生产者与消费者的关系，有一个micro-op queue作为消费队列**

在一些传统的方法中，我们可以通过如下的方式模拟阻塞(Stalls)给系统带来的损耗：
![image](https://github.com/spevenhe/Study/assets/42630862/554b4af1-86e7-4fdf-9cea-9e917f919777)

例如分支预测失败会带来两个时钟的损耗，通过多少次分支预测失败这个事件就可以计算损耗的时钟数。然后再算上别的性能事件，就可以得到一共损耗的时钟周期了。这个方法看起来是好用的，但是在现代乱序CPU中，由于如下的一些问题，这个方法失效了：

超标量带来的Stalls误差(Stalls overlap)：在现代超标量处理器中，CPU可以同时发射或执行多条指令，一条流水线阻塞不代表整条流水线阻塞；
投机执行(Speculative Execution)：投机执行的指令效率比不投机指令的效率还要低；
阻塞叠加影响：多个阻塞会互相叠加；
遗漏预定义事件：通过累加计算损耗就需要目标事件集合，但是这个集合可能会出现遗漏的情况

# top down

Top-Down分析方法是一种自顶向下，逐步分解的性能分析方法。 就是按照决策树的方式去遍历

Topdown 只能帮助解决 CPU 受限问题。 如果瓶颈在其他地方，则必须使用其他方法。 非 CPU 瓶颈可以是网络、阻塞延迟（例如同步）、磁盘 IO、显卡等。 Topdown 根据 CPU 的 Pipeline 处理流程将性能瓶颈分成了四类：Frontend Bound，Bad Speculation，Retiring，Backend Bound。

2.1 Frontend Bound
Frontend 是 Pipeline 的第一部分，包括了获取分支预测器预测的下一个地址、获取缓存行、解析指令、解码 Backend 之后可执行的微操作等步骤。 因此 Frontend Bound 占比大的话，意味着获取指令、解析指令等核心部分是程序的瓶颈所在，也就是 ITLB miss 或 Icache miss 等情况。

2.2 Bad Speculation
错误推演反映了由于错误推演而浪费的 slot 资源。 其中包括两部分：1、发射最终不会 retired 的微指令； 2、由于从早期的错误推演中恢复而导致 pipeline 被阻塞。 例如，在错误推演的分支下发射的微指令导致的 slot 资源浪费属于此分类。

2.3 Retiring
Ritiring 反映了发射的指令真正被架构执行最后 Retired 的占比，一个理想的程序是 Retiring 的占比为100%。因此 Retiring 占比越高，反映了程序被浪费的资源越少，效率越高。

2.4 Backend Bound
Backend 是 pipeline 的后半部分，Frontend Bound 反映了指令被解码成微指令被发射后，在指令执行阶段没有交付到下一阶段的情况，造成 Backend Bound 的情况可能：data-cache miss 或者 Divider 过载导致的阻塞等。Backend bound 可以分为 core bound 和 memory bound。

core bound
Core Bound 有点棘手。 它的停顿可能表现为较短的执行饥饿期，或次优的执行端口利用率：一个长延迟的除法操作可能会序列化执行，而服务于特定类型微指令的执行端口的压力可能表现为一个周期中使用的端口数量很少。Core Bound 问题通常可以通过更好的代码生成来缓解。 例如，一系列相关算术运算将被归类为 Core Bound。 编译器可以通过更好的指令调度来缓解这种情况。 矢量化也可以缓解 Core Bound 问题。

memory bound
Memory Bound 对应于与内存子系统相关的执行停顿。 这些停顿通常表现为执行单元在短时间后变得饥饿，就像加载丢失所有缓存的情况一样。

![image](https://github.com/spevenhe/Study/assets/42630862/86b18860-f340-4d5d-b9e6-66de75ad4c2b)

![image](https://github.com/spevenhe/Study/assets/42630862/bf9d0803-6913-4324-81f9-9a2bc44907f9)
