
# flink状态分类

![image](https://user-images.githubusercontent.com/42630862/141883160-6e4ab5ad-58ff-4a54-8446-e3ed8635aa9a.png)

![image](https://user-images.githubusercontent.com/42630862/141883198-eb90afac-8448-468a-930a-9005f1cd54ea.png)

![image](https://user-images.githubusercontent.com/42630862/141883229-adb92fee-4f83-41d6-b39b-8769bdc82897.png)

![image](https://user-images.githubusercontent.com/42630862/141883990-ce9407a4-0121-46bb-aa06-67d38a0b8532.png)

![image](https://user-images.githubusercontent.com/42630862/141884018-3d6cd985-9f09-40ed-9a3b-c5b3fd8c9d2b.png)

![image](https://user-images.githubusercontent.com/42630862/141885065-80b71e5a-89ec-484a-a808-e2ec8314ade2.png)


# flink容错之恢复策略

Flink 容错机制主要有**作业执行的容错**以及**守护进程的容错两方面**，
前者包括 Flink runtime 的 ExecutionGraph 和 Execution 的容错，
后者则包括 JobManager 和 TaskManager 的容错。

Flink 的错误恢复机制分为多个级别，即 Execution 级别的 Failover 策略和 ExecutionGraph 级别的 Job Restart 策略。当出现错误时，Flink 会先尝试触发范围小的错误恢复机制，如果仍处理不了才会升级为更大范围的错误恢复机制，具体可以用下面的序列图来表达（其中省略了Exection 和 ExecutionGraph 的非关键状态转换）。
Exection：subtask 级别
ExecutionGraph：job级别

![image](https://user-images.githubusercontent.com/42630862/141889823-47de164a-5bdf-45a5-b0be-d887af69975e.png)

当 Task 发生错误，TaskManager 会通过 RPC 通知 JobManager，后者将对应 Execution 的状态转为 failed 并触发 Failover 策略。如果符合 Failover 策略，JobManager 会重启 Execution，否则升级为 ExecutionGraph 的失败。ExecutionGraph 失败则进入 failing 的状态，由 Restart 策略决定其重启（restarting 状态）还是异常退出（failed 状态）。

## Task Failover 策略

RestartAll: 重启全部 Task，是恢复作业一致性的最安全策略，会在其他 Failover 策略失败时作为保底策略使用。目前是默认的 Task Failover 策略。
![image](https://user-images.githubusercontent.com/42630862/141890046-5c8d87e4-4ae0-40a7-af81-fc3daef9fa85.png)

RestartPipelinedRegionStrategy: 重启错误 Task 所在 Region 的全部 Task。Task Region 是由 Task 的数据传输决定的，有数据传输的 Task 会被放在同一个 Region，而不同 Region 之间没有数据交换。

RestartIndividualStrategy: 恢复单个 Task。因为如果该 Task 没有包含数据源，这会导致它不能重流数据而导致一部分数据丢失。考虑到至少提供准确一次的投递语义，这个策略的使用范围比较有限，只应用于 Task 间没有数据传输的作业。不过也有部分业务场景可能需要这种 at-most-once 的投递语义，比如对延迟敏感而对数据一致性要求相对低的推荐系统。


## Job Restart 策略
如果 Task 错误最终触发了 Full Restart，此时 Job Restart 策略将会控制是否需要恢复作业。Flink 提供三种 Job 具体的 Restart Strategy。

FixedDelayRestartStrategy: 允许指定次数内的 Execution 失败，如果超过该次数则导致 Job 失败。FixedDelayRestartStrategy 重启可以设置一定的延迟，以减少频繁重试对外部系统带来的负载和不必要的错误日志。目前 FixedDelayRestartStrategy 是默认的 Restart Strategy。

FailureRateRestartStrategy: 允许在指定时间窗口内的指定次数内的 Execution 失败，如果超过这个频率则导致 Job 失败。同样地，FailureRateRestartStrategy 也可以设置一定的重启延迟。

NoRestartStrategy: 在 Execution 失败时直接让 Job 失败。

目前的 Restart Strategy 可以基本满足“自动重启挂掉的作业”这样的简单需求，然而并没有区分作业出错的原因，这导致可能会对不可恢复的错误（比如用户代码抛出的 NPE 或者某些操作报 Permission Denied）进行不必要的重试，进一步的后果是没有第一时间退出，可能导致用户没有及时发现问题，其外对于资源来说也是一种浪费，最后还可能导致一些副作用（比如有些 at-leaset-once 的操作被执行多次）。

对此，社区在 1.7 版本引入了 Exception 的分类[5]，具体会将 Runtime 抛出的 Exception 分为以下几类:

NonRecoverableError: 不可恢复的错误。不对此类错误进行重试。

PartitionDataMissingError: 当前 Task 读不到上游 Task 的某些数据，需要上游 Task 重跑和重发数据。

EnvironmentError: 执行环境的错误，通常是 Flink 以外的问题，比如机器问题、依赖问题。这种错误的一个明显特征是会在某些机器上执行成功，但在另外一些机器上执行失败。Flink 后续可以引入黑名单机器来更聪明地进行 Task 调度以暂时避免这类问题的影响。

RecoverableError: 可恢复错误。不属于上述类型的错误都暂设为可恢复的。

其实这个分类会应用于 Task Failover 策略和 Job Restart 策略，不过目前只有后者会分类处理，而且 Job Restart 策略对 Flink 作业的稳定性影响显然更大，因此放在这个地方讲。值得注意的是，截至目前（1.8 版本）这个分类只处于很初级的阶段，像 NonRecoverable 只包含了作业 State 命名冲突等少数几个内部错误，而 PartitionDataMissingError 和 EnvironmentError 还未有应用，所以绝大多数的错误仍是 RecoverableError。
如果 Task 错误最终触发了 Full Restart，此时 Job Restart 策略将会控制是否需要恢复作业。Flink 提供三种 Job 具体的 Restart Strategy。

FixedDelayRestartStrategy: 允许指定次数内的 Execution 失败，如果超过该次数则导致 Job 失败。FixedDelayRestartStrategy 重启可以设置一定的延迟，以减少频繁重试对外部系统带来的负载和不必要的错误日志。目前 FixedDelayRestartStrategy 是默认的 Restart Strategy。

FailureRateRestartStrategy: 允许在指定时间窗口内的指定次数内的 Execution 失败，如果超过这个频率则导致 Job 失败。同样地，FailureRateRestartStrategy 也可以设置一定的重启延迟。

NoRestartStrategy: 在 Execution 失败时直接让 Job 失败。

目前的 Restart Strategy 可以基本满足“自动重启挂掉的作业”这样的简单需求，然而并没有区分作业出错的原因，这导致可能会对不可恢复的错误（比如用户代码抛出的 NPE 或者某些操作报 Permission Denied）进行不必要的重试，进一步的后果是没有第一时间退出，可能导致用户没有及时发现问题，其外对于资源来说也是一种浪费，最后还可能导致一些副作用（比如有些 at-leaset-once 的操作被执行多次）。

对此，社区在 1.7 版本引入了 Exception 的分类[5]，具体会将 Runtime 抛出的 Exception 分为以下几类:

NonRecoverableError: 不可恢复的错误。不对此类错误进行重试。

PartitionDataMissingError: 当前 Task 读不到上游 Task 的某些数据，需要上游 Task 重跑和重发数据。

EnvironmentError: 执行环境的错误，通常是 Flink 以外的问题，比如机器问题、依赖问题。这种错误的一个明显特征是会在某些机器上执行成功，但在另外一些机器上执行失败。Flink 后续可以引入黑名单机器来更聪明地进行 Task 调度以暂时避免这类问题的影响。

RecoverableError: 可恢复错误。不属于上述类型的错误都暂设为可恢复的。
     
其实这个分类会应用于 Task Failover 策略和 Job Restart 策略，不过目前只有后者会分类处理，而且 Job Restart 策略对 Flink 作业的稳定性影响显然更大，因此放在这个地方讲。值得注意的是，截至目前（1.8 版本）这个分类只处于很初级的阶段，像 NonRecoverable 只包含了作业 State 命名冲突等少数几个内部错误，而 PartitionDataMissingError 和 EnvironmentError 还未有应用，所以绝大多数的错误仍是 RecoverableError。


# flink容错之checkpoints
exactly onec 实现

![image](https://user-images.githubusercontent.com/42630862/158554336-6e22cdc3-7e75-4f8e-9740-43e932a7043e.png)
 
![image](https://user-images.githubusercontent.com/42630862/158729012-0a164857-aa52-42ee-9324-11d561f4b8c3.png)

## 检查点算法实现

 ![image](https://user-images.githubusercontent.com/42630862/158729656-50bc2b44-31dd-49c6-abd5-83f0d1b25cd2.png)

 ![image](https://user-images.githubusercontent.com/42630862/158731374-ba0a72b3-7c9e-4f92-b6b3-3c1bae5314f0.png)


## 分布式检查点算法

![image](https://user-images.githubusercontent.com/42630862/158787198-9178762a-02dc-40ed-9734-11efdf994d45.png)

barrier独立于 数据，是jm发送的特殊标识，tm各个算子收到后将储存checkpoint到statebackend

![image](https://user-images.githubusercontent.com/42630862/158789126-50beac62-4f8d-42d1-bea7-067c9cdf1c74.png)

 



