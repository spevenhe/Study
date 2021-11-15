## 参考http://alexstocks.github.io/html/rocksdb.html
## https://github.com/facebook/rocksdb/wiki/RocksDB-Overview
## rocksd性能测试https://github.com/facebook/rocksdb/wiki/Performance-Benchmarks
RocksDB 是一个快速存储系统，它会充分挖掘 Flash or RAM 硬件的读写特性，支持单个 KV 的读写以及批量读写。RocksDB 自身采用的一些数据结构如 LSM/SKIPLIST 等结构使得其有读放大、写放大和空间使用放大的问题。

基本概念： 同步写入内存，异步刷入磁盘，采用lsm tree

LSM-Tree(Log-Structured-Merge-Tree) 
![image](https://user-images.githubusercontent.com/42630862/141769690-77685206-1ae4-4387-9ab2-ac49ff787654.png)
LSM 大致结构如上图所示。LSM 树而且通过批量存储技术规避磁盘随机写入问题。 LSM 树的设计思想非常朴素, 它的原理是把一颗大树拆分成N棵小树， 它首先写入到内存中（内存没有寻道速度的问题，随机写的性能得到大幅提升），在内存中构建一颗有序小树，随着小树越来越大，内存的小树会flush到磁盘上。磁盘中的树定期可以做 merge 操作，合并成一棵大树，以优化读性能【读数据的过程可能需要从内存 memtable 到磁盘 sstfile 读取多次，称之为读放大】。RocksDB 的 LSM 体现在多 level 文件格式上，最热最新的数据尽在 L0 层，数据在内存中，最冷最老的数据尽在 LN 层，数据在磁盘或者固态盘上。

LSM-Tree(Log-Structured-Merge-Tree)
LSM从命名上看，容易望文生义成一个具体的数据结构，一个tree。但LSM并不是一个具体的数据结构，也不是一个tree。LSM是一个数据结构的概念，是一个数据结构的设计思想。实际上，要是给LSM的命名断句，Log和Structured这两个词是合并在一起的，LSM-Tree应该断句成Log-Structured、Merge、Tree三个词汇，这三个词汇分别对应以下三点LSM的关键性质：

1. 将数据形成Log-Structured：在将数据写入LSM内存结构之前，先记录log。这样LSM就可以将有易失性的内存看做永久性存储器。并且信任内存上的数据，等到内存容量达到threshold再集体写入磁盘。将数据形成Log-Structured，也是将整体存储结构转换成了“内存(in-memory)”存储结构。
2. 将所有磁盘上数据不组织成一个整体索引结构，而组织成有序的文件集：因为磁盘随机读写比顺序读写慢3个数量级，LSM尽量将磁盘读写转换成顺序读写。将磁盘上的数据组织成B树这样的一个整体索引结构，虽然查找很高效，但是面对随机读写，由于大量寻道导致其性能不佳。而LSM用了一种很有趣的方法，将所有数据不组织成一个整体索引结构，而组织成有序的文件集。每次LSM面对磁盘写，将数据写入一个或几个新生成的文件，顺序写入且不能修改其他文件，这样就将随机读写转换成了顺序读写。LSM将一次性集体写入的文件作为一个level，磁盘上划分多level，level与level之间互相隔离。这就形成了，以写入数据时间线形成的逻辑上、而非物理上的层级结构，这也就是为什么LSM被命名为”tree“，但不是“tree”。
3. 将数据按key排序，在合并不同file、level上的数据时类似merge-join：如果一直保持生成新的文件，不仅写入会造成冗余空间，而且也会大量降低读的性能。所以要高效的、周期性合并不同file、level。而如果数据是乱序的，根本做不到高效合并。所以LSM要将数据按key排序，在合并不同file、level上的数据时类似 merge-join。

RocksDB is a storage engine library of key-value store interface where keys and values are arbitrary byte streams. RocksDB organizes all data in sorted order and the common operations are **Get(key), NewIterator(), Put(key, val), Delete(key), and SingleDelete(key).**

The three basic constructs of RocksDB are **memtable, sstfile and logfile**. The memtable is an in-memory data structure - new writes are inserted into the memtable and are optionally written to the logfile (aka. Write Ahead Log(WAL)). The logfile is a sequentially-written file on storage. When the memtable fills up, it is flushed to a sstfile on storage and the corresponding logfile can be safely deleted. The data in an sstfile is sorted to facilitate easy lookup of keys.
![image](https://user-images.githubusercontent.com/42630862/141772206-8c92ce64-3199-4f76-90b0-44374352016f.png)



