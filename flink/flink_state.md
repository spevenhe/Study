
# 仅仅针对于1.12版本

## state backends是什么？
在启动 CheckPoint 机制时，状态会随着 CheckPoint 而持久化，以防止数据丢失、保障恢复时的一致性。 状态内部的存储格式、状态在 CheckPoint 时如何持久化以及持久化在哪里均取决于选择的 State Backend。

## 仅注意rocksdbstatebackend

RocksDBStateBackend 需要配置一个文件系统的 URL （类型、地址、路径），例如：”hdfs://namenode:40010/flink/checkpoints” 或 “file:///data/flink/checkpoints”。

RocksDBStateBackend 将正在运行中的状态数据保存在 RocksDB 数据库中，RocksDB 数据库默认将数据存储在 TaskManager 的数据目录。 CheckPoint 时，整个 RocksDB 数据库被 checkpoint 到配置的文件系统目录中。 少量的元数据信息存储到 JobManager 的内存中（高可用模式下，将其存储到 CheckPoint 的元数据文件中）。

RocksDBStateBackend 只支持异步快照。

RocksDBStateBackend 的限制：

由于 RocksDB 的 JNI API 构建在 byte[] 数据结构之上, 所以每个 key 和 value 最大支持 2^31 字节。 重要信息: RocksDB 合并操作的状态（例如：ListState）累积数据量大小可以超过 2^31 字节，但是会在下一次获取数据时失败。这是当前 RocksDB JNI 的限制。
RocksDBStateBackend 的适用场景：

状态非常大、窗口非常长、key/value 状态非常大的 Job。
所有高可用的场景。
注意，你可以保留的状态大小仅受磁盘空间的限制。与状态存储在内存中的 FsStateBackend 相比，RocksDBStateBackend 允许存储非常大的状态。 然而，这也意味着使用 RocksDBStateBackend 将会使应用程序的最大吞吐量降低。 所有的读写都必须序列化、反序列化操作，这个比基于堆内存的 state backend 的效率要低很多。

## rocksdb 使用：

``StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

env.setStateBackend(new RocksDBStateBackend("hdfs:///fink-checkpoints", true));
# 开启增量
``

<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-statebackend-rocksdb_2.11</artifactId>
    <version>1.12.3</version>
    <scope>provided</scope>
</dependency>
