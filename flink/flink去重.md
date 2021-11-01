
## 方式1 mapstate


一个mapstate负责过滤，一个valuestate负责计数。增量


## 方式2 sql 全量
为了与离线分析保持一致的分析语义，Flink SQL 中提供了distinct去重方式，使用方式：
SELECT DISTINCT devId FROM pv

表示对设备ID进行去重，得到一个明细结果，那么我们在使用distinct来统计去重结果通常有两种方式, 仍然以统计每日网站uv为例。
第一种方式

SELECT datatime,count(DISTINCT devId) FROM pv group by datatime

该语义表示计算网页每日的uv数量，其内部核心实现主要依靠DistinctAccumulator与CountAccumulator，DistinctAccumulator 内部包含一个map结构，key 表示的是distinct的字段，value表示重复的计数，CountAccumulator就是一个计数器的作用，这两部分都是作为动态生成聚合函数的中间结果accumulator,透过之前的聚合函数的分析可知中间结果是存储在状态里面的，也就是容错并且具有一致性语义的
其处理流程是：

将devId 添加到对应的DistinctAccumulator对象中，首先会判断map中是否存在该devId, 不存在则插入map中并且将对应value记1，并且返回True;存在则将对应的value+1更新到map中，并且返回False
只有当返回True时才会对CountAccumulator做累加1的操作,以此达到计数目的


第二种方式

select count(*),datatime from(

select distinct devId,datatime from pv ) a

group by datatime

内部是一个对devId,datatime 进行distinct的计算，在flink内部会转换为以devId,datatime进行分组的流并且进行聚合操作，在内部会动态生成一个聚合函数，该聚合函数createAccumulators方法生成的是一个Row(0) 的accumulator 对象，其accumulate方法是一个空实现，也就是该聚合函数每次聚合之后返回的结果都是Row(0),通过之前对sql中聚合函数的分析(可查看GroupAggProcessFunction函数源码)， 如果聚合函数处理前后得到的值相同那么可能会不发送该条结果也可能发送一条撤回一条新增的结果，但是其最终的效果是不会影响下游计算的，在这里我们简单理解为在处理相同的devId,datatime不会向下游发送数据即可,也就是每一对devId,datatime只会向下游发送一次数据；

外部就是一个简单的按照时间维度的计数计算，由于内部每一组devId,datatime 只会发送一次数据到外部，那么外部对应datatime维度的每一个devId都是唯一的一次计数，得到的结果就是我们需要的去重计数结果。

两种方式对比

这两种方式最终都能得到相同的结果，但是经过分析其在内部实现上差异还是比较大，第一种在分组上选择datatime ，内部使用的累加器DistinctAccumulator 每一个datatime都会与之对应一个对象，在该维度上所有的设备id, 都会存储在该累加器对象的map中，而第二种选择首先细化分组，使用datatime+devId分开存储，然后外部使用时间维度进行计数，简单归纳就是：
第一种: datatime->Value{devI1,devId2..}
第二种: datatime+devId->row(0)
聚合函数中accumulator 是存储在ValueState中的，第二种方式的key会比第一种方式数量上多很多，但是其ValueState占用空间却小很多，而在实际中我们通常会选择Rocksdb方式作为状态后端，rocksdb中value大小是有上限的，第一种方式很容易到达上限，那么使用第二种方式会更加合适；
这两种方式都是全量保存设备数据的，会消耗很大的存储空间，但是我们的计算通常是带有时间属性的，那么可以通过配置StreamQueryConfig设置状态ttl。
