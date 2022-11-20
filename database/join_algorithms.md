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

## 3.1 hash function
**对于double float, 就要避免collison rate, 对int bigint简易的很容易perfect hashing**
**目前andy pavlo talked to 的 database system 跟company, just pick one hash function,对所有的data benchmark,
nobody nobody nobody使用adaptive hash function/hash table schema**

![image](https://user-images.githubusercontent.com/42630862/202907257-9abdc353-9c8c-4a3d-8d18-ed0e8395b356.png)


![image](https://user-images.githubusercontent.com/42630862/202906942-88ef8900-4c19-449d-887a-55784ea71f2c.png)

## 3.2 HASHING SCHEMES

![image](https://user-images.githubusercontent.com/42630862/202907758-ee0b8625-ac9f-45e6-8577-d67ee4f6fcbe.png)

### 3.2.1 CHAINED HASHING 适合

加入bloom filter 做优化， bloom filter 储存在bucket pointer 内，因为intel 使用48bit 作为实际的pointer?
![image](https://user-images.githubusercontent.com/42630862/202907888-97e8f45d-050b-4ffc-bf06-f47713ad6900.png)

### 3.2.2 LINEAR PROBE HASHING 不适合
![image](https://user-images.githubusercontent.com/42630862/202908100-2b983e9e-ac8d-42cd-aa4d-a341a9e51132.png)
![image](https://user-images.githubusercontent.com/42630862/202908242-1b5818b3-6248-4d10-abc2-1a80e73084b7.png)

### 3.2.3 ROBIN HOOD HASHING 不适合

![image](https://user-images.githubusercontent.com/42630862/202908359-385a2604-b11a-4381-8ff4-d9d555262c23.png)

![image](https://user-images.githubusercontent.com/42630862/202908334-1ac69e45-f559-4f75-8602-f394175c4217.png)

### 3.2.4 ROBIN HOOD HASHING 不适合
### 3.2.5 CUCKOO HASHING 不适合
![image](https://user-images.githubusercontent.com/42630862/202908563-c48fb978-368b-4997-8382-add09fb94a1f.png)

# 4 PROBE PHASE
![image](https://user-images.githubusercontent.com/42630862/202910429-65bf80f0-0dad-4f58-8688-bc572c6c0cf1.png)

![image](https://user-images.githubusercontent.com/42630862/202910434-b35c48fb-ece8-4882-bd06-ce5581e6b877.png)

# benchmark

andy 说很难超过 hyper 的 linear probe hashing

![image](https://user-images.githubusercontent.com/42630862/202910807-222e8a96-9f5c-4034-b208-92e1c87787e8.png)

![image](https://user-images.githubusercontent.com/42630862/202910845-3a1adab7-6c91-440e-8256-c3b59742bf5d.png)


![image](https://user-images.githubusercontent.com/42630862/202910937-d464c3d7-0350-4d8f-9142-0d4b94d849f5.png)

![image](https://user-images.githubusercontent.com/42630862/202911479-3aff80b9-e8f6-40f6-b69f-f0e920dbfb15.png)


![image](https://user-images.githubusercontent.com/42630862/202911545-31fff878-a143-44af-a034-8a5074aabeb7.png)

![image](https://user-images.githubusercontent.com/42630862/202911742-40bcdced-0bc3-4340-980b-151f67c8874e.png)

