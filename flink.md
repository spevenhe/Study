# flink 集群
### 1 flink 任务提交流程
对于session与per-job
首先在客户端本地执行main（），下载资源，然后将资源与jobGraph一起提交到集群中

session模式：
优：资源使用
劣：一个job挂了，带走一个tm，所有使用那个tm的job都会被影响

per-job:
资源隔离，分散jm压力：生产使用


### 2. flink sql
 1. flinksql 是什么：flink 是最高层的，批流统一
 2. flink sql 中必须包含时间字段

特殊性：group Aggregation:
flink sql 的group by之后，输出的结果是不断更新的，相同的字段会更新输出，比如先a,1  a,2  a,3..... **需要sink到能够更新结果的sink里，如mysql，es，hbase**
