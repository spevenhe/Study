# join 向量化

## 理论基础 
某种类型的operator fusion
## 基本思路 
1. hash 值计算
2. build 过程
3. probe 过程

### 1. hash向量化的 计算hash值 vectorhasher 可以参考velox
### 2. build过程，
2.1 储存 array of struct：hash value 跟 key column other column 存在一起，

因为可能存在冲突，无法使用向量化build方式

2.2 储存 struct of array, hash value 跟 columns 分开储存，可以在column 储存用向量化
在储存结束后 向量化算key 然后 非向量化 用hashvalue + 地址偏移的方式构建hashtable

### 3. probe 过程
向量化集中优化点
1. linear probing 一类hashtable. 垂直向量化 或 水平向量化
垂直向量化实现较为复杂：参考Rethinking SIMD Vectorization for In-Memory Databases
水平向量化 用一个batch 记录没有match的，下次将没有match的hash index+1

2. chained 一类hashtable. 水平向量化
向量化实现较为方便，但是如果chained 的深度不够，可能吞吐上不去
