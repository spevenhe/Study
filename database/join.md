# join 语法
![image](https://user-images.githubusercontent.com/42630862/179871229-b081f9ad-ca4d-448e-9154-8d34d64d759d.png)



# 1 hash join 原理
## 1.1 mysql的hash join
### 1.1.1 内存能放下所有的hash row

![image](https://user-images.githubusercontent.com/42630862/179874656-4c17047b-410f-4d0c-81a8-857553038565.png)

![image](https://user-images.githubusercontent.com/42630862/179874670-95ed6ddb-7bdb-4219-8059-e358f60bd618.png)

### 1.1.2 内存放不下， 利用disk 分治
![image](https://user-images.githubusercontent.com/42630862/179877191-e684d448-8fba-4e9e-9afc-20871cd5e499.png)

![image](https://user-images.githubusercontent.com/42630862/179877156-da4e9228-657b-41a8-9bc2-a2130f91fb3a.png)
![image](https://user-images.githubusercontent.com/42630862/179877165-cc0013cb-5a7c-4a7f-aa8b-fe6fcef7f2af.png)


## 1.2 tidb hash join
https://pingcap.com/zh/blog/tidb-source-code-reading-9

### 1.2.1 构建 Hash Join 执行器

TiDB 首先会根据 SQL 来构建相应的 Logic Plan；

然后将 Logic Plan 转成 Physical Plan，这里是转成 PhysicalHashJoin 作为 Physical Plan；

通过比较 Physical Plan 的代价，最后选择一个代价最小的 Physical Plan 构建执行器 executor；

代价计算 根据数据量做一个估算


## 1.3 Cider join
This library will provide a set of API and mechinism which is easy for user to use. 
1. User should prepare a CiderBatch (which will be entire right table to be hashed) before call ComipleModule::compile method, 
2. and pass this CiderBatch as input of ComipleModule::compile method. Cider library will handle hash table build and other stuff internally. 
3. after that, user will get a compilationResult, and should build a CiderRuntimeModule. 
4. When user call CiderRuntimeModule::processNextBatch(), just as usual, set left table(should convert to CiderBatch) as first input parameter, and user could get result for this batch join. Please check example use case. 

Non-join columns -> Table
join columns -> HashTable

reuse buffer and multi thread

## 1.4 omnisci hash join
eliminate the inner loop from a loop join and replace it with a hash table lookup


### hash join buffers
A hash join buffer can have up to four sections which are **_located consecutively_** in memory:

**keys offsets counts** are reletaed

Keys
is an array containing the hashable keys from the key/value pairs to be stored in the hash table.

Offsets
 is an array containing integer indexes into the Payloads section.
 
Counts
is an array containing the integer sizes of the subarrays stored in the Payloads section.

Payloads
 is an array of subarrays, with each subarray containing one or more row ID integer references for rows in the one of the join tables. 


### Kinds of Hash Join


