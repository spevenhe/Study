# flink 集群
### 1 flink 任务提交流程
对于session与per-job
首先在客户端本地执行main（），下载资源，然后将资源与jobGraph一起提交到集群中
![image](https://user-images.githubusercontent.com/42630862/110566809-8367fe80-818b-11eb-8eb5-897e08a763c1.png)

session模式：
优：资源使用
劣：一个job挂了，带走一个tm，所有使用那个tm的job都会被影响

per-job:
资源隔离，分散jm压力：生产使用

### 2 flink 聚合区别
![image](https://user-images.githubusercontent.com/42630862/110566516-1bb1b380-818b-11eb-9f98-c09ac03c00cd.png)

group aggregation的输出的结果是不断更新的，相同的字段会更新输出，比如先a,1  a,2  a,3..... **需要sink到能够更新结果的sink里，如mysql，es，hbase**。group 作业都要配置ttl，去清除状态。


### 2. flink sql
 1. flinksql 是什么：flink 是最高层的，批流统一
 2. flink sql 中必须包含时间字段
 3. flink sql 支持的数据类型：
 
 ![image](https://user-images.githubusercontent.com/42630862/110748462-b63cf000-827a-11eb-99b6-5e590cf2ff69.png)

3.1 注意点：timestamp（p） p代表秒的精度

  4. flink sql kafka
注意点 flink sql kafka 必填format或者value format，因为不是所有kafka message 都有key。一般用 ‘format’=‘json’，然后在 key.fields中能指定 key的对象，value也能指定value的对象，两者重叠使用key.fields_prefix具体例子见https://ci.apache.org/projects/flink/flink-docs-release-1.12/zh/dev/table/connectors/kafka.html

4.1 需要用刀flink-json包，sql-client 内置



#### 2.1 sql-client

1. 使用UDF 继承类scalarFunction，打包，扔到lib，查看show functions

2. flink-sql demo 

小技巧1： proctime as PROCTIME() as 可以作为生成一个虚拟列
小技巧2： over window 是
所有的聚合必须定义到同一个窗口中，即相同的分区、排序和区间。当前仅支持 PRECEDING (无界或有界) 到 CURRENT ROW 范围内的窗口、FOLLOWING 所描述的区间并未支持，ORDER BY 必须指定于单个的时间属性

kafka字段数限制，java数组字段数的限制

10个流的join：每次两个

使用blink

sql 接Kafka，解析ddl，生成consumer

嵌套的json ROW<>




flink sql 与kafka partrition 相同的并发度

激活catalog 持久化catalog

 ### 3 flink 调优
 #### 3.1 Flink 如何权衡吞吐与延迟的关系
 正在写入的 buffer 肯定不能被 Netty 消费到，只有写完的 buffer 才能被消费到。在 Flink 中有三种情况会认为 buffer 写完了，可以被 Netty  Server 消费：

buffer 写满了

buffer 超时了

遇到特殊的 Event，例如：Checkpoint barrier

 通过tm间 netty的buffer来设置：
 Flink 1.10 及以后的版本直接通过配置参数 execution.buffer-timeout: 100ms
 
 
 #### 3.2 Flink 并发度大会导致CPU资源消耗高
 
 生产环境很多任务的并发远大于 1000，所以造成很多 buffer 仅仅只缓存 1 条数据就被 timeout 策略触发发送给下游 Task。每条数据做为 1 个 buffer，每秒处理 1 万条数据，则后台线程每秒需要 flush 1 万个 buffer 到 NettyServer，从而大量消耗 CPU。

不仅是发送方效率降低，下游的 Subtask B 接受数据的效率也会降低。每秒接受 1 万的 buffer，每个 buffer 里 1 条数据。大量的小 buffer，大量的读取小数据，消耗大量的 CPU 资源。

 #### 3.3 Flink cpu调优

测试条件：业务逻辑简单，keyBy 后不做任何业务处理，并发 1200。

因为没有业务处理，所以从业务角度来讲，CPU 应该主要消耗在序列化和反序列化 protobuf。从引擎角度来讲，TM 之间数据 flush、传输需要消耗不少的 CPU。

基于上述条件，仅调节 bufferTimeout 参数，观察单个 TM 的平均 CPU 消耗。

bufferTimeout = 100ms 时，TM 平均 CPU 使用 0.59 core
bufferTimeout = 1s 时，TM 平均 CPU 使用 0.39 core，CPU 节省了 33%
bufferTimeout = 10s 时，TM 平均 CPU 使用 0.33 core，CPU 节省了 44%
 
 

