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

### 3. flink sql 底层

api 层次
![image](https://user-images.githubusercontent.com/42630862/114370878-89019b80-9bb2-11eb-8994-7555c05c3e1e.png)

catalog 架构
![image](https://user-images.githubusercontent.com/42630862/114371567-32e12800-9bb3-11eb-8bfd-ac31349561a4.png)


用户可以切换catalog，catalog在yaml中显式定义 1.9 ？


![image](https://user-images.githubusercontent.com/42630862/114371734-60c66c80-9bb3-11eb-91e5-a5e9377fbba6.png)
目前有两个地方可以使用ddl 1：table environment 2： sql client

sql 向 job graph的转换：
![image](https://user-images.githubusercontent.com/42630862/114372191-da5e5a80-9bb3-11eb-8171-48f0144fd0c5.png)

1. sql 转换为 sqlNode

2. 在functionCatalog 看一看有没有udf

3. 转化为operation dag

4. operation dag 转化为 relNode dag

5. relnode dag涉及到优化器，在优化器里被优化

6. 优化后转化为execNode dag

7. 然后最后会被转化为jobgraph


**分段优化的例子 **


![image](https://user-images.githubusercontent.com/42630862/114373268-03cbb600-9bb5-11eb-9249-3b41b7cf4a92.png)


![image](https://user-images.githubusercontent.com/42630862/114373422-21991b00-9bb5-11eb-9ac2-fe1465af8cee.png)

**agg 的分类**
 ![image](https://user-images.githubusercontent.com/42630862/114387969-ab50e480-9bc5-11eb-9d2f-9967850b282d.png)


group agg 优化

![image](https://user-images.githubusercontent.com/42630862/114388079-cfacc100-9bc5-11eb-9854-ec4173958902.png)

** 重要**

![image](https://user-images.githubusercontent.com/42630862/114388436-42b63780-9bc6-11eb-93aa-ec89cdd08eac.png)

![image](https://user-images.githubusercontent.com/42630862/114388520-5d88ac00-9bc6-11eb-955e-9cb65063118a.png)

![image](https://user-images.githubusercontent.com/42630862/114388843-d38d1300-9bc6-11eb-9e62-eaf730878843.png)

![image](https://user-images.githubusercontent.com/42630862/114389028-12bb6400-9bc7-11eb-9320-85bb0eb8f849.png)




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





#### 2.2 flink sql 数据打宽
![image](https://user-images.githubusercontent.com/42630862/114491137-a2edbd80-9c48-11eb-88b4-34ed6a89d3d7.png)

双流join 因为数据是无穷保存的，一定要指定ttl


![image](https://user-images.githubusercontent.com/42630862/114491310-037cfa80-9c49-11eb-91c3-e0cc0c72d8b5.png)

stae保留时间，只保留区间数据

 #### 2.3.2 时态表 temporal table
![image](https://user-images.githubusercontent.com/42630862/114491612-7a19f800-9c49-11eb-8e6a-8c95e5c09749.png)

### 2.4 flink sql 语句例子

foundation tips

1. similar

SELECT 
  log_time,
  request_line,
  status_code,
  size
FROM server_logs
WHERE 
  status_code SIMILAR TO '4[0-9][0-9]'
  
2. temporary table 临时表

临时表不会储存在catalog中，只存在于 current session

对于不会使用的表结构，在flink sql中建为临时表

3. 对于时间乱序的数据 可以使用 order by + group by timewindow

4. 使用view 非物理储存，来简化复杂脚本

5. 向多个表写入数据

6. 复杂sql 可使用sql regrep Oracle相关api

聚合与分析
1. group by tumble ,对每一个 key都输出统计值，可以与其他双重group，tumble 固定窗口滑动

适用于key少的情况

2. 加入watermark 处理乱序

3. session window 按间隔统计数据

4. OVER
OVER window aggregates compute an aggregated value for every input row over a range of ordered rows. In contrast to GROUP BY aggregates, OVER aggregates do not reduce the number of result rows to a single row for every group. Instead, OVER aggregates produce an aggregated value for every input row.

over 的意思是来一条就输出一次，并不是每个时间范围输出一次，适用于key很多的情况

5. top N  有特定语法结构，须保持一致，不然blink解析不了

6. 去重 特殊的topN top1

7. 链式事件时间窗口：比如求1小时平均与 1min平均，可以创建两个view，其中1min得view根据1h得view  创建，然后将1min得window插入1h得window 中去，通过另一个表、view创建view，可以形成链式

flink sql 内置函数

1.   time
CREATE TABLE subscriptions ( 
    id STRING,
    start_date INT,
    end_date INT,
    payment_expiration TIMESTAMP(3)
) WITH (
  'connector' = 'faker',
  'fields.id.expression' = '#{Internet.uuid}', 
  'fields.start_date.expression' = '#{number.numberBetween ''1576141834'',''1607764234''}',
  'fields.end_date.expression' = '#{number.numberBetween ''1609060234'',''1639300234''}',
  'fields.payment_expiration.expression' = '#{date.future ''365'',''DAYS''}'
);

SELECT 
  id,
  TO_TIMESTAMP(FROM_UNIXTIME(start_date)) AS start_date,
  TO_TIMESTAMP(FROM_UNIXTIME(end_date)) AS end_date,
  DATE_FORMAT(payment_expiration,'YYYYww') AS exp_yweek,
  EXTRACT(DAY FROM payment_expiration) AS exp_day,     --same as DAYOFMONTH(ts)
  EXTRACT(MONTH FROM payment_expiration) AS exp_month, --same as MONTH(ts)
  EXTRACT(YEAR FROM payment_expiration) AS exp_year    --same as YEAR(ts)
FROM subscriptions
WHERE 
  TIMESTAMPDIFF(DAY,CURRENT_TIMESTAMP,payment_expiration) < 30;


2.  union all


### 2.5 flink watermark 配置

1. rowtime 代表事件时间



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
 
 #### 3.4 Flink 并行度优化
 Flink 中算子链接（chain）起来的条件
Flink 在内部会将多个算子串在一起作为一个 operator chain（执行链）来执行，每个执行链会在 TaskManager 上的一个独立线程中执行，这样不仅可以减少线程的数量及线程切换带来的资源消耗，还能降低数据在算子之间传输序列化与反序列化带来的消耗。

举个例子，拿一个 Flink Job （算子的并行度都设置为 5）生成的 StreamGraph JSON 渲染出来的执行流程图如下图所示。

![image](https://user-images.githubusercontent.com/42630862/112783835-d1816b00-9082-11eb-9980-fa1f619995dc.png)


提交到 Flink UI 上的 JobGraph 如下图所示。




![image](https://user-images.githubusercontent.com/42630862/112783829-ccbcb700-9082-11eb-8063-26d2518a81f0.png)


可以看到 Flink 它内部将三个算子（source、filter、sink）都串成在一个执行链里。但是我们修改一下 filter 这个算子的并行度为 4，我们再次提交到 Flink UI 上运行，效果如下图所示。
![image](https://user-images.githubusercontent.com/42630862/112783816-c3334f00-9082-11eb-87b6-62c06678edbd.png)
你会发现它竟然拆分成三个了，我们继续将 sink 的并行度也修改成 4，继续打包运行后的效果如下图所示。
![image](https://user-images.githubusercontent.com/42630862/112783811-c0385e80-9082-11eb-99a9-138d96628053.png)

图片
神奇不，它变成了 2 个了，将 filter 和 sink 算子串在一起了执行了。经过简单的测试，我们可以发现其实如果想要把两个不一样的算子串在一起执行确实还不是那么简单的，的确，它背后的条件可是比较复杂的，这里笔者给出源码出来，感兴趣的可以独自阅读下源码。

public static boolean isChainable(StreamEdge edge, StreamGraph streamGraph) {
    //获取StreamEdge的源和目标StreamNode
 StreamNode upStreamVertex = edge.getSourceVertex();
 StreamNode downStreamVertex = edge.getTargetVertex();
 
    //获取源和目标StreamNode中的StreamOperator
 StreamOperator<?> headOperator = upStreamVertex.getOperator();
 StreamOperator<?> outOperator = downStreamVertex.getOperator();

 return downStreamVertex.getInEdges().size() == 1
   && outOperator != null
   && headOperator != null
   && upStreamVertex.isSameSlotSharingGroup(downStreamVertex)
   && outOperator.getChainingStrategy() == ChainingStrategy.ALWAYS
   && (headOperator.getChainingStrategy() == ChainingStrategy.HEAD ||
    headOperator.getChainingStrategy() == ChainingStrategy.ALWAYS)
   && (edge.getPartitioner() instanceof ForwardPartitioner)
   && upStreamVertex.getParallelism() == downStreamVertex.getParallelism()
   && streamGraph.isChainingEnabled();
}
从源码最后的 return 可以看出它有九个条件：

下游节点只有一个输入
下游节点的操作符不为 null
上游节点的操作符不为 null
上下游节点在一个槽位共享组（slotsharinggroup）内，默认是 default
下游节点的连接策略是 ALWAYS（可以与上下游节点连接）
上游节点的连接策略是 HEAD 或者 ALWAYS
edge 的分区函数是 ForwardPartitioner 的实例（没有 keyby 等操作）
上下游节点的并行度相等
允许进行节点连接操作（默认允许）
所以看到上面的这九个条件，你是不是在想如果我们代码能够合理的写好，那么就有可能会将不同的算子串在一个执行链中，这样也就可以提高代码的执行效率了。
 
 
 #### 3.5 Flink 内存调优
 ###### 配置内存
 
操作场景

Flink是依赖内存计算，计算过程中内存不够对Flink的执行效率影响很大。可以通过监控GC（Garbage Collection），评估内存使用及剩余情况来判断内存是否变成性能瓶颈，并根据情况优化。
监控节点进程的YARN的Container GC日志，如果频繁出现Full GC，需要优化GC。
GC的配置：在客户端的"conf/flink-conf.yaml"配置文件中，在“env.java.opts”配置项中添加参数：

-Xloggc:<LOG_DIR>/gc.log 

-XX:+PrintGCDetails 

-XX:-OmitStackTraceInFastThrow 

-XX:+PrintGCTimeStamps 

-XX:+PrintGCDateStamps 

-XX:+UseGCLogFileRotation 

-XX:NumberOfGCLogFiles=20 

-XX:GCLogFileSize=20M

此处默认已经添加GC日志。

操作步骤
优化GC

调整老年代和新生代的比值。在客户端的“conf/flink-conf.yaml”配置文件中，在“env.java.opts”配置项中添加参数：“-XX:NewRatio”。如“ -XX:NewRatio=2”，则表示老年代与新生代的比值为2:1，新生代占整个堆空间的1/3，老年代占2/3。
开发Flink应用程序时，优化DataStream的数据分区或分组操作。

当分区导致数据倾斜时，需要考虑优化分区。避免非并行度操作，有些对DataStream的操作会导致无法并行，例如WindowAll。keyBy尽量不要使用String。



##### 3.配置进程参数
操作场景

Flink on YARN模式下，有JobManager和TaskManager两种进程。在任务调度和运行的过程中，JobManager和TaskManager承担了很大的责任。

因而JobManager和TaskManager的参数配置对Flink应用的执行有着很大的影响意义。用户可通过如下操作对Flink集群性能做优化。

操作步骤

1.配置JobManager内存

JobManager负责任务的调度，以及TaskManager、RM之间的消息通信。当任务数变多，任务平行度增大时，JobManager内存都需要相应增大。您可以根据实际任务数量的多少，为JobManager设置一个合适的内存。
在使用yarn-session命令时，添加“-jm MEM”参数设置内存。

在使用yarn-cluster命令时，添加“-yjm MEM”参数设置内存。

2.配置TaskManager个数

每个TaskManager每个核同时能跑一个task，所以增加了TaskManager的个数相当于增大了任务的并发度。在资源充足的情况下，可以相应增加TaskManager的个数，以提高运行效率。
在使用yarn-session命令时，添加“-n NUM”参数设置TaskManager个数。

在使用yarn-cluster命令时，添加“-yn NUM”参数设置TaskManager个数。

3.配置TaskManager Slot数


每个TaskManager多个核同时能跑多个task，相当于增大了任务的并发度。但是由于所有核共用TaskManager的内存，所以要在内存和核数之间做好平衡。
在使用yarn-session命令时，添加“-s NUM”参数设置SLOT数。

在使用yarn-cluster命令时，添加“-ys NUM”参数设置SLOT数。

4.配置TaskManager内存


TaskManager的内存主要用于任务执行、通信等。当一个任务很大的时候，可能需要较多资源，因而内存也可以做相应的增加。
将在使用yarn-sesion命令时，添加“-tm MEM”参数设置内存。

将在使用yarn-cluster命令时，添加“-ytm MEM”参数设置内存。

##### 配置netty网络通信
操作场景

Flink通信主要依赖netty网络，所以在Flink应用执行过程中，netty的设置尤为重要，网络通信的好坏直接决定着数据交换的速度以及任务执行的效率。
操作步骤

以下配置均可在客户端的“conf/flink-conf.yaml”配置文件中进行修改适配，默认已经是相对较优解，请谨慎修改，防止性能下降。

•“taskmanager.network.netty.num-arenas”：默认是“taskmanager.numberOfTaskSlots”，表示netty的域的数量。

•“taskmanager.network.netty.server.numThreads”和“taskmanager.network.netty.client.numThreads”：默认是“taskmanager.numberOfTaskSlots”，表示netty的客户端和服务端的线程数目设置。 

•“taskmanager.network.netty.client.connectTimeoutSec”：默认是120s，表示taskmanager的客户端连接超时的时间。

“taskmanager.network.netty.sendReceiveBufferSize”：默认是系统缓冲区大小(cat /proc/sys/net/ipv4/tcp _ [rw]mem) ，一般为4MB，表示netty的发送和接收的缓冲区大小。

“taskmanager.network.netty.transport”：默认为“nio”方式，表示netty的传输方式，有“nio”和“epoll”两种方式。
 
 
 ##### 解决数据倾斜
 
当数据发生倾斜（某一部分数据量特别大），虽然没有GC（Gabage Collection，垃圾回收），但是task执行时间严重不一致。

需要重新设计key，以更小粒度的key使得task大小合理化。

修改并行度。

调用rebalance操作，使数据分区均匀。

缓冲区超时设置

由于task在执行过程中存在数据通过网络进行交换，数据在不同服务器之间传递的缓冲区超时时间可以通过setBufferTimeout进行设置。

当设置“setBufferTimeout(-1)”，会等待缓冲区满之后才会刷新，使其达到最大吞吐量；当设置“setBufferTimeout(0)”时，可以最小化延迟，数据一旦接收到就会刷新；当设置“setBufferTimeout”大于0时，缓冲区会在该时间之后超时，然后进行缓冲区的刷新。示例可以参考如下：env.setBufferTimeout(timeoutMillis); env.generateSequence(1,10).map(new MyMapper()).setBufferTimeout(timeoutMillis);
 




 ### 4flink 问题汇总
 
**资源不足导致 container 被 kill**

`The assigned slot container_container编号 was removed.`
Flink App 抛出此类异常，通过查看日志，一般就是某一个 Flink App 内存占用大，导致 TaskManager（在 Yarn 上就是 Container ）被Kill 掉。

但是并不是所有的情况都是这个原因，还需要进一步看 yarn 的日志（ 查看 yarn 任务日志：yarn logs -applicationId  -appOwner），如果代码写的没问题，就确实是资源不够了，其实 1G Slot 跑多个Task（ Slot Group Share ）其实挺容易出现的。

因此有两种选择，可以根据具体情况，权衡选择一个。

将该 Flink App 调度在 Per Slot 内存更大的集群上。
通过 slotSharingGroup("xxx") ，减少 Slot 中共享 Task 的个数



# FLINK ML

![image](https://user-images.githubusercontent.com/42630862/114337007-d534e780-9b82-11eb-8743-4ecbaf3ea7ce.png)

![image](https://user-images.githubusercontent.com/42630862/114352866-f4417280-9b9e-11eb-9a0a-5cf1f9e96f32.png)


### 目前思路

##### 大的思路是在 离线模型的基础上，在用实时的数据对模型坐微调

**1. 在历史数据，按照日期为单位，通过长时间范围判断当天的交易额，然后预测当天的交易额，情况是否正常，此为 按天 维度的**

**2. 历史数据存在hdfs，每次将同星期，同月今日，与去年今日数据拿出，作为训练集，训练出按 小时的 或者10min 维度的预测**

**3. kafka数据直接进入模型做出预测**

**4. kakfa同时还有一部分数据进入当日数据集，然后将当日数据集与当日时间作为数据进行训练，实时更新当日模型，具体为 通过当天的走势预测接下来的交易，思路有2 ： 1是将今日数据与往日数据结合起来，进行训练，但是提高今日数据的权重，二是将今日数据作为loss function，只训练 往日数据**


![image](https://user-images.githubusercontent.com/42630862/114365559-2b1e8500-9bad-11eb-95c7-3ff43099f88f.png)


1. transformer 是对数据预处理，以及在线推理的抽象
2. 是数据训练模型

![image](https://user-images.githubusercontent.com/42630862/114366295-e6471e00-9bad-11eb-929d-27c36b3d6e18.png)



机器学习使用点：
新商户注册信息 预测新商户流水
使用KNN
讲现有商户先kmeans
根据kmeans，注册资本，行业，信息，分类成多类
使用knn将新商户分类，做出流水预测


通过老商户正常流水，识别异常交易行为
普通预测



### flink 内存模型
1.10 以上版本
![image](https://user-images.githubusercontent.com/42630862/136755831-3be5f981-2c29-4f56-95a1-d507cad40b57.png)


Flink JVM 进程的*进程总内存（Total Process Memory）*包含了由 Flink 应用使用的内存（Flink 总内存）以及由运行 Flink 的 JVM 使用的内存。 Flink 总内存（Total Flink Memory）包括 JVM 堆内存（Heap Memory）和堆外内存（Off-Heap Memory）。 其中堆外内存包括直接内存（Direct Memory）和本地内存（Native Memory）。




