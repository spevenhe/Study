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

