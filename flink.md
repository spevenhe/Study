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

 


