# flink memory

## 配置总内存 #

Flink JVM 进程的*进程总内存（Total Process Memory）*包含了由 Flink 应用使用的内存（Flink 总内存）以及由运行 Flink 的 JVM 使用的内存。 Flink 总内存（Total Flink Memory）包括 JVM 堆内存（Heap Memory）和堆外内存（Off-Heap Memory）。 其中堆外内存包括直接内存（Direct Memory）和本地内存（Native Memory）。


![image](https://user-images.githubusercontent.com/42630862/136757083-b37b0b35-4892-49fe-b647-4fc991adcdc4.png)
![image](https://user-images.githubusercontent.com/42630862/136757119-b986ddc6-6d08-4c43-a8aa-565443bc0071.png)

![image](https://user-images.githubusercontent.com/42630862/136757318-45e5b291-e661-4466-bbc9-023f2903facd.png)
![image](https://user-images.githubusercontent.com/42630862/136757338-abd134b2-5cbd-43c1-80b1-bb5c5be7d81b.png)

### 容器（Container）的内存配置 #
在容器化部署模式（Containerized Deployment）下（Kubernetes、Yarn 或 Mesos），建议配置进程总内存（taskmanager.memory.process.size 或者 jobmanager.memory.process.size）。 该配置参数用于指定分配给 Flink JVM 进程的总内存，也就是需要申请的容器大小。

提示 如果配置了 Flink 总内存，Flink 会自动加上 JVM 相关的内存部分，根据推算出的进程总内存大小申请容器。

注意： 如果 Flink 或者用户代码分配超过容器大小的非托管的堆外（本地）内存    **任务堆外内存（Task Off-heap Memory）	taskmanager.memory.task.off-heap.size	用于 Flink 应用的算子及用户代码的堆外内存（直接内存或本地内存）。**

，部署环境可能会杀掉超用内存的容器，造成作业执行失败。

批处理作业的内存配置 #
Flink 批处理算子使用托管内存
**Managed memory	taskmanager.memory.managed.size
taskmanager.memory.managed.fraction	Native memory managed by Flink, reserved for sorting, hash tables, caching of intermediate results and RocksDB state backend**

 来提高处理效率。 算子运行时，部分操作可以直接在原始数据上进行，而无需将数据反序列化成 Java 对象。 这意味着托管内存对应用的性能具有实质上的影响。 因此 Flink 会在不超过其配置限额的前提下，尽可能分配更多的托管内存。 Flink 明确知道可以使用的内存大小，因此可以有效避免 OutOfMemoryError 的发生。 当托管内存不足时，Flink 会优雅地将数据落盘。
