# flink 基础运行的架构
![image](https://user-images.githubusercontent.com/42630862/127332138-f9ce1e95-0b70-4ae0-93ef-a4b11bcfee22.png)
## 基于yarn
![image](https://user-images.githubusercontent.com/42630862/127332489-7905086b-27a5-449d-bc03-32e68130b161.png)

## 并行度

![image](https://user-images.githubusercontent.com/42630862/127332740-91173f19-9bc1-47bd-8d31-183cb87b2ca1.png)
![image](https://user-images.githubusercontent.com/42630862/127332893-ab3aca42-5626-433c-a86d-048c8624f6df.png)
推荐一个slot一个cpu核心，几核cpu，给tm设置几个slot
![image](https://user-images.githubusercontent.com/42630862/127333016-d89400ed-bd5a-469b-bbc5-82ac5070de2e.png)
子任务可以slot共享，可以一个slot运行多个子任务，**前提在于前后不一样的任务才能在同一个slot，并行的子任务不能在一个slot**

对于slot可以在代码里设置 算子 的 slotsharinggroup，来确定算子在不在一个slot共享组
**后面slot不设置的话，就跟前面一个组是一样的**

## DataFlow

![image](https://user-images.githubusercontent.com/42630862/127339549-4354f261-b17c-40a5-9d53-af178b6f7de1.png)

![image](https://user-images.githubusercontent.com/42630862/127351279-022bd509-636d-4b7e-a7b8-4bdb6252cb2b.png)


## 窗口聚合函数
窗口函数（window function

•
window function 定义 了要对窗口中收集的数据做的计算 操作

•
可以分为两类

1
增量聚合函数（ i ncremental aggregation functions

•
每条数据到来就进行计算，保持一个简单的状态

•
ReduceFunction, AggregateFunction

其中reducefunction 与 aggregatefunction 最大的区别在于输出类型的不同

2
全窗口函数（ f ull window functions

•
先把窗口所有数据收集起来，等到计算的时候 会 遍历所有数据

•
ProcessWindowFunction WindowFunction


## watermark
那么此时出现一个问题，一旦出现乱序，如果只根据
eventTime 决定 window 的
运行，我们不能明确数据是否全部到位，但又不能无限期的等下去，此时必须要有
个机制来保证一个特定的时间后，必须触发 window 去 进行计算了，这个特别的机
制，就是 Watermark 。

⚫ Watermark 是一种衡量 Event Time 进展的机制 。

⚫ Watermark 是用于处理乱序事件的 ，而正确的处理乱序事件，通常用
Watermark 机制结合 window 来实现。

⚫ 数据流中的 Watermark 用于表示 timestamp 小于 Watermark 的数据，都已经
到达了，因此， window 的执行也是由 Watermark 触发的。

⚫ Watermark 可以理解成一个延迟触发机制，我们可以设置 Watermark 的延时
时长 t ，每次系统会校验已经到达的数据中最大 的 max E vent T ime ，然后认定 eventTime
小于 max E vent T ime t 的所有数据都已经到达，如果有窗口的停止时间等于
max E vent T ime t ，那么这个窗口被触发执行。

**Watermark**
就是触发前一窗口的“关窗时间”，一旦触发关门那么以当前时刻
为准在窗口范围内的所有所有数据都会收入窗中。
只要没有达到水位那么不管现实中的时间推进了多久都不会触发关窗。

