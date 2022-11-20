# 1 hash join
![image](https://user-images.githubusercontent.com/42630862/202904036-b7f0ca12-d8de-49d3-9365-7a790dbfda09.png)


## partition hash join
![image](https://user-images.githubusercontent.com/42630862/202073655-2adbd7ad-4f92-43d0-8d79-1d2f94963a19.png)

**when hashtable size are too large, use recursive hash function**

![image](https://user-images.githubusercontent.com/42630862/202073672-2fd4ac9a-1a65-4b7c-8165-ebf96c194417.png)

**For unknown hashtable size, use dynamic hashtable**

![image](https://user-images.githubusercontent.com/42630862/202073744-e32a9210-401d-4e7c-bd8b-7a37785795b7.png)

# 2 PARTITION PHASE

NSM row store

DSM column store

![image](https://user-images.githubusercontent.com/42630862/202904141-62f71803-6f4f-4939-b270-f453b5b290e3.png)

![image](https://user-images.githubusercontent.com/42630862/202904266-6156a9cf-82ac-4ee0-b167-2d898d70ba38.png)

## 2.1 NON-BLOCKING PARTITIONING
![image](https://user-images.githubusercontent.com/42630862/202904315-b4a81199-335a-42e1-93eb-efa069bef76d.png)

### Approach #1: Shared Partitions

![image](https://user-images.githubusercontent.com/42630862/202904500-de67a1f1-2706-4093-9bc6-4d4397649d26.png)


### Approach #2: Private Partitions
**对于row store, 最后一步开销很大，对于column store 最后一步开销较小**

![image](https://user-images.githubusercontent.com/42630862/202904555-b065302c-4b4d-49c7-94f5-96e4670bff5b.png)

## 2.2 BLOCKING PARTITIONING/RADIX PARTITIONING
# 实际中很少有人使用，基本上都是学术原型

![image](https://user-images.githubusercontent.com/42630862/202904863-64dc8ac2-4d34-485c-b4e9-2a639ebe60aa.png)

**使用prefix sum 提前分配好buff空间，现在cpu 对于计算radix 非常得快

![image](https://user-images.githubusercontent.com/42630862/202905187-0539ae0d-9081-4c04-82a6-448cfa83a584.png)

![image](https://user-images.githubusercontent.com/42630862/202905149-651862ff-41b7-4f55-aacd-09c4fd821bc8.png)

![image](https://user-images.githubusercontent.com/42630862/202905252-51922ca1-ed98-46d2-8b00-8dd0f93efcdf.png)

# 3 BUILD PHASE

![image](https://user-images.githubusercontent.com/42630862/202906812-0ad03b87-27cf-492e-85c9-788407547da1.png)

**对于double float, 就要避免collison rate, 对int bigint简易的很容易perfect hashing**
**目前andy pavlo talked to 的 database system 跟company, just pick one hash function,对所有的data benchmark,
nobody nobody nobody使用adaptive hash function**
![image](https://user-images.githubusercontent.com/42630862/202907257-9abdc353-9c8c-4a3d-8d18-ed0e8395b356.png)


![image](https://user-images.githubusercontent.com/42630862/202906942-88ef8900-4c19-449d-887a-55784ea71f2c.png)




