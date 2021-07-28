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




