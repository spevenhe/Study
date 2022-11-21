## 缩写

**NUMA: Non-uniform memory access**

Non-uniform memory access is a computer memory design used in multiprocessing, where the memory access time depends on the memory location relative to the processor. Under NUMA, a processor can access its own local memory faster than non-local memory

## 论文对象集

we analyze the impact of various joins on an (unchanged) TPC-H query

# 一些推荐总结

### For primary key columns

it is common sense to use automatically generated integer
IDs. This leads to a dense key domain of integers. For this
distribution a simple array rather than a hash table may be a
surprisingly good choice.

### for non-dense distributions, usually on primary key
a compressed array like the recently suggested [17] may be a good choice to avoid using a full-blown hash table.
[17] R Barber G Lohman I Pandis, V Raman R Sidle, G Attaluri N
Chainani S Lightstone, and D Sharpe. Memory-Efficient Hash Joins.
PVLDB, 8(4):353–364, 2014.

# cider hashtable 实现
1. 不能使用 radix hash, 首先 radix hashing 需要知道全部的数据，velox 是一批批传下来的，
2. 基本上不能使用 partition
3. build 的 hash table schema，使用 chained + bloom filter
4. 对于velox 下发的，目前是 private partition + merge 
5. 对于 data 超过 memory 大小，使用递归 partition hash join/ grace hash join
6. 对于 multi core, 充分利用性能，使用partition hash  join

