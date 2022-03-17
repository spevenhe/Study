# flink 状态一致性
![image](https://user-images.githubusercontent.com/42630862/158844505-f958bf79-f8e6-41cf-97d1-760682343882.png)

## 状态一致性：exactly once一致
 
对于source 读取两次，但是状态一致，也属于exactly once

## sink exactly once
###1 幂等写入
代表 redis

###2 事务写入
 
![image](https://user-images.githubusercontent.com/42630862/158844524-712d6df9-602e-4f6a-94ef-8c5e70fa77a7.png)

## 预写日志
 ![image](https://user-images.githubusercontent.com/42630862/158844541-ca66acc6-2508-4832-94c3-3c4f2f8496f6.png)
![image](https://user-images.githubusercontent.com/42630862/158844553-2e2fe88b-318c-4d1f-a19f-5da778c11cbc.png)

 ![image](https://user-images.githubusercontent.com/42630862/158844572-fe087585-0989-4319-a96b-bab31800b261.png)

 
 # kafka sink exactly-once

 ![image](https://user-images.githubusercontent.com/42630862/158844584-7945a0c4-0d94-4f34-8962-14591d4754b3.png)

**kafka 设置隔离级别，必须commited才能消费，不commited不能消费**”
