## refer:
http://flink.apache.org/2021/10/26/sort-shuffle-part1.html

https://flink.apache.org/2021/10/26/sort-shuffle-part2.html

## data passed between operators

1. hash shuffle
   直接把数据输出到不同的文件，每个文件都能作为分区的数据
   
2. sort shuffle
   先将说有的数据写在一起，然后利用sorting 将数据聚类到不同的数据分区甚至数据keys
   
 
## hash shuffle 的缺点

1. 稳定性：开启太多文件，对文件系统有压力，同事会耗尽file descriptor 
2. peffomrance 对小文件涉及了随机读写

sorted shuffle 还会减少网络缓存

hash shuffle会导致大型作业启动有问题，connection reset by peer 与 netty threads influence netwoek

## How to use this new feature
The sort-based blocking shuffle is introduced mainly for large-scale batch jobs but it also works well for batch jobs with low parallelism.

The sort-based blocking shuffle is not enabled by default. You can enable it by setting the **taskmanager.network.sort-shuffle.min-parallelism**
config option to a smaller value. This means that for parallelism smaller than this threshold, the hash-based blocking shuffle will be used, otherwise, the sort-based blocking shuffle will be used (it has no influence on streaming applications). Setting this option to 1 will disable the hash-based blocking shuffle.

## 实现
Sort-Spill-Merge 的方式被分布式计算系统广泛采纳以达到这一目标，首先将数据写入内存缓冲区，当内存缓冲区填满后对数据进行排序，排序后的数据被写出到一个文件中，这样总的文件数量是：（总数据量 / 内存缓冲区大小），从而文件数量被减少。当所有数据写出完成后，将产生的文件合并成一个文件，从而进一步减少文件数量并增大每个数据分区的大小（有利于顺序读取）。


相比于其他系统的实现，Flink 的实现有一个重要的不同，即 Flink 始终向同一个文件中不断追加数据，而不会写多个文件再进行合并，这样的好处始终只有一个文件，文件数量实现了最小化。

类似于其他分布式计算系统中 sort-shuffle 的实现，Flink 利用一块固定大小的内存缓冲区进行数据的缓存与排序。这块内存缓冲区的大小是与并发无关的，从而使得上游 shuffle 数据写所需要的内存缓冲区大小与并发解耦。结合另一个内存管理方面的优化 FLINK-16428 可以同时实现下游 shuffle 数据读取的内存缓冲区消耗并发无关化，从而可以减少大规模批作业的内存缓冲区消耗。


1. 内存数据排序

在排序算法上，我们选择了复杂度较低的 bucket-sort。具体而言，每条序列化后的数据前面都会被插入一个 16 字节的元数据。包括 4 字节的长度、4 字节的数据类型以及 8 字节的指向同一数据分区中下一条数据的指针。结构如下图所示：

![image](https://user-images.githubusercontent.com/42630862/141747522-b10c4edf-a63e-4f95-a0e1-0a09ca0517dc.png)



当从缓冲区中读取数据时，只需要按照每个数据分区的链式索引结构就可以读取到属于这个数据分区的所有数据，并且这些数据保持了数据写入时的顺序。这样按照数据分区的顺序读取所有的数据就可以达到按照数据分区排序的目标。

2.2 文件储存

如前所述，每个并行任务产生的 shuffle 数据会被写到一个物理文件中。每个物理文件包含多个数据区块（data region），每个数据区块由数据缓冲区的一次 sort-spill 生成。在每个数据区块中，所有属于不同数据分区（data partition，由下游计算节点不同并行任务消费）的数据按照数据分区的序号顺序进行排序聚合。下图展示了 shuffle 数据文件的详细结构。其中（R1，R2，R3）是 3 个不同的数据区块，分别对应 3 次数据的 sort-spill 写出。每个数据块中有 3 个不同的数据分区，分别将由（C1，C2，C3）3 个不同的并行消费任务进行读取。也就是说数据 B1.1，B2.1 及 B3.1 将由 C1 处理，数据 B1.2，B2.2 及 B3.2 将由 C2 处理，而数据 B1.3，B2.3 及 B3.3 将由 C3 处理。
![image](https://user-images.githubusercontent.com/42630862/141747721-5788b1e2-20ae-43ad-8927-d0446b1b3d0c.png)

类似于其他的分布式处理系统实现，在 Flink 中，每个数据文件还对应一个索引文件。索引文件用来在读取时为每个消费者索引属于它的数据（data partition）。索引文件包含和数据文件相同的 data region，在每个 data region 中有与 data partition 相同数量的索引项，每个索引项包含两个部分，分别对应到数据文件的偏移量以及数据的长度。作为一个优化。Flink 为每个索引文件缓存最多 4M 的索引数据。数据文件与索引文件的对应关系如下：

![image](https://user-images.githubusercontent.com/42630862/141747779-b62d73eb-6bfc-4c02-8c00-7c9e6c9219db.png)




