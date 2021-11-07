# flink memory

## flink 内存模型概述 #

Flink JVM 进程的*进程总内存（Total Process Memory）*包含了由 Flink 应用使用的内存（Flink 总内存）以及由运行 Flink 的 JVM 使用的内存。 Flink 总内存（Total Flink Memory）包括 JVM 堆内存（Heap Memory）和堆外内存（Off-Heap Memory）。 其中堆外内存包括直接内存（Direct Memory）和本地内存（Native Memory）。


![image](https://user-images.githubusercontent.com/42630862/136757083-b37b0b35-4892-49fe-b647-4fc991adcdc4.png)
![image](https://user-images.githubusercontent.com/42630862/136760562-2e2ca92d-ec97-4515-b147-13b8a708fe7a.png)
![image](https://user-images.githubusercontent.com/42630862/136757338-abd134b2-5cbd-43c1-80b1-bb5c5be7d81b.png)





## 配置flink 进程内存#
![image](https://user-images.githubusercontent.com/42630862/136757318-45e5b291-e661-4466-bbc9-023f2903facd.png)
![image](https://user-images.githubusercontent.com/42630862/136757119-b986ddc6-6d08-4c43-a8aa-565443bc0071.png)


## 配置 taskmanager 的内存

### 1 配置堆内存和托管内存 #
如配置总内存中所述，另一种配置 Flink 内存的方式是同时设置任务堆内存和托管内存。 通过这种方式，用户可以更好地掌控用于 Flink 任务的 JVM 堆内存及 Flink 的托管内存大小。

Flink 会根据默认值或其他配置参数自动调整剩余内存部分的大小。 关于各内存部分的更多细节，请参考相关文档。

**提示 如果已经明确设置了任务堆内存和托管内存，建议不要再设置进程总内存或 Flink 总内存，否则可能会造成内存配置冲突。**


方式1 直接配置总内存taskmanager.memory.flink.size 与jobmanager.memory.flink.size

方式2 配置 task heap memory and managed memory

Managed memory is managed by Flink and is allocated as native memory (off-heap). The following workloads use managed memory:



task heap : If you want to guarantee that a certain amount of JVM Heap is available for your user code, you can set the task heap memory explicitly (taskmanager.memory.task.heap.size). It will be added to the JVM Heap size and will be dedicated to Flink’s operators running the user code.

managed memory: configured explicitly via taskmanager.memory.managed.size


Streaming jobs can use it for RocksDB state backend.

Both streaming and batch jobs can use it for sorting, hash tables, caching of intermediate results.

Both streaming and batch jobs can use it for executing User Defined Functions in Python processes.

### 2 Configure Off-heap Memory (direct or native) #
The off-heap memory which is allocated by user code should be accounted for in task off-heap memory (taskmanager.memory.task.off-heap.size).也就是用户自己的代码使用的堆内存

可以通过 taskmanager.memory.task.off-heap.size 指定


### 容器（Container）的内存配置 #
**在容器化部署模式（Containerized Deployment）下（Kubernetes、Yarn 或 Mesos），建议配置进程总内存（taskmanager.memory.process.size 或者 jobmanager.memory.process.size）。 该配置参数用于指定分配给 Flink JVM 进程的总内存，也就是需要申请的容器大小**。

提示 如果配置了 Flink 总内存，Flink 会自动加上 JVM 相关的内存部分，根据推算出的进程总内存大小申请容器。

注意： 如果 Flink 或者用户代码分配超过容器大小的非托管的堆外（本地）内存    **任务堆外内存（Task Off-heap Memory）	taskmanager.memory.task.off-heap.size	用于 Flink 应用的算子及用户代码的堆外内存（直接内存或本地内存）。**

，部署环境可能会杀掉超用内存的容器，造成作业执行失败。


### rocksDB 内存配置
RocksDBStateBackend 使用本地内存。 默认情况下，RocksDB 会限制其内存用量不超过用户配置的托管内存。 因此，使用这种方式存储状态时，配置足够多的托管内存是十分重要的。 如果你关闭了 RocksDB 的内存控制，那么在容器化部署模式下如果 RocksDB 分配的内存超出了申请容器的大小（进程总内存），可能会造成 TaskExecutor 被部署环境杀掉。 请同时参考如何调整 RocksDB 内存以及 state.backend.rocksdb.memory.managed。


批处理作业的内存配置 #
Flink 批处理算子使用托管内存
**托管内存（Managed memory）	taskmanager.memory.managed.size
taskmanager.memory.managed.fraction	由 Flink 管理的用于排序、哈希表、缓存中间结果及 RocksDB State Backend 的本地内存。**

 来提高处理效率。 算子运行时，部分操作可以直接在原始数据上进行，而无需将数据反序列化成 Java 对象。 这意味着托管内存对应用的性能具有实质上的影响。 因此 Flink 会在不超过其配置限额的前提下，尽可能分配更多的托管内存。 Flink 明确知道可以使用的内存大小，因此可以有效避免 OutOfMemoryError 的发生。 当托管内存不足时，Flink 会优雅地将数据落盘。

## jobmanager  内存配置
![image](https://user-images.githubusercontent.com/42630862/140650586-9858e38c-2693-47dd-83b1-aa6d16d268a7.png)




