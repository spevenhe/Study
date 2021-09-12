## 核心在于倒排索引 基于Lucene
### 倒排索引
1 关键词
term 关键词

posting list 每个term对应的doc的id的合集，

 id 的范围，在存储数据的时候，在每一个 shard 里面，ES 会将数据存入不同的 segment，这是一个比 shard 更小的分片单位，这些 segment 会定期合并。
在每一个 segment 里面都会保存最多 2^31 个文档，每个文档被分配一个唯一的 id，从 0 到 (2^31)-1。

![image](https://user-images.githubusercontent.com/42630862/132990078-6397fb67-5de5-4b16-82c1-84eb46929c92.png)

2 内部索引结构

term dictionary：于是乎就有了 term dictionary，ES 为了能快速查找到 term，将所有的 term 排了一个序，二分法查找。

term index：但是问题又来了，你觉得 Term Dictionary 应该放在哪里？肯定是放在内存里面吧？磁盘 io 那么慢。就像 MySQL 索引就是存在内存里面了。

Term index 从数据结构上分类算是一个“Trie 树”，也就是我们常说的字典树。这是一种专门处理字符串匹配的数据结构，用来解决在一组字符串集合中快速查找某个字符串的问题。

这棵树不会包含所有的 term，它包含的是 term 的一些前缀（这也是字典树的使用场景，公共前缀）。通过 term index 可以快速地定位到 term dictionary 的某个 offset，然后从这个位置再往后顺序查找。就想右边这个图所表示的。（怎么样，像不像我们查英文字典，我们定位 S 开头的第一个单词，或者定位到 Sh 开头的第一个单词，然后再往后顺序查询）
lucene 在这里还做了两点优化，一是 term dictionary 在磁盘上面是分 block 保存的，一个 block 内部利用公共前缀压缩，比如都是 Ab 开头的单词就可以把 Ab 省去。二是 term index 在内存中是以 FST（finite state transducers）的数据结构保存的。


![image](https://user-images.githubusercontent.com/42630862/132990189-64306fa7-dddb-490d-a30d-4dac84072fac.png)

### posting list的技巧

1.压缩

1.1 Frame of Reference

![image](https://user-images.githubusercontent.com/42630862/132990378-0c62e473-a82f-44f7-9a99-e0f54c077d55.png)

![image](https://user-images.githubusercontent.com/42630862/132990385-883253dd-b060-4d4a-be81-53f0f59e8fcc.png)

1.2 Roaring Bitmaps

假设有这么一个数组，我们第一个压缩的思路是什么？用位的方式来表示，每个文档对应其中的一位，也就是我们常说的位图，bitmap。

![image](https://user-images.githubusercontent.com/42630862/132991010-90ec189b-7b50-48b4-b0ca-40c8e6423b1d.png)

但是，位图有个很明显的缺点，不管业务中实际的元素基数有多少，它占用的内存空间都恒定不变。也就是说不适用于稀疏存储。业内对于稀疏位图也有很多成熟的压缩方案，lucene 采用的就是roaring bitmaps。
我这里用简单的方式描述一下这个压缩过程是怎么样，
![image](https://user-images.githubusercontent.com/42630862/132991127-eb3cf341-b3d6-4029-b4b1-9e0c0cdb172f.png)

将 doc id 拆成高 16 位，低 16 位。对高位进行聚合 (以高位做 key，value 为有相同高位的所有低位数组)，根据低位的数据量 (不同高位聚合出的低位数组长度不相同)，使用不同的 container(数据结构) 存储。

len<4096 ArrayContainer 直接存值
len>=4096 BitmapContainer 使用 bitmap 存储

分界线的来源：value 的最大总数是为2^16=65536. 假设以 bitmap 方式存储需要 65536bit=8kb,而直接存值的方式，一个值 2 byte，4K 个总共需要2byte*4K=8kb。所以当 value 总量 <4k 时,使用直接存值的方式更节省空间。
![image](https://user-images.githubusercontent.com/42630862/132991135-58fb9972-0789-4e82-9972-fe797cc7a3ae.png)
      
空间压缩主要体现在:

高位聚合 (假设数据中有 100w 个高位相同的值,原先需要 100w2byte,现在只要 12byte)
低位压缩

缺点就在于位操作的速度相对于原生的 bitmap 会有影响。


2. 联合查询

先讲简单的，如果查询有 filter cache，那就是直接拿 filter cache 来做计算，也就是说位图来做 AND 或者 OR 的计算。
如果查询的 filter 没有缓存，那么就用 skip list 的方式去遍历磁盘上的 postings list。

![image](https://user-images.githubusercontent.com/42630862/132992991-afc91e2e-4cdc-4ddb-ac06-ead4e58e89af.png)

以上是三个 posting list。我们现在需要把它们用 AND 的关系合并，得出 posting list 的交集。首先选择最短的 posting list，逐个在另外两个 posting list 中查找看是否存在，最后得到交集的结果。遍历的过程可以跳过一些元素，比如我们遍历到绿色的 13 的时候，就可以跳过蓝色的 3 了，因为 3 比 13 要小。
用 skip list 还会带来一个好处，还记得前面说的吗，postings list 在磁盘里面是采用 FOR 的编码方式存储的

会把所有的文档分成很多个 block，每个 block 正好包含 256 个文档，然后单独对每个文档进行增量编码，计算出存储这个 block 里面所有文档最多需要多少位来保存每个 id，并且把这个位数作为头信息（header）放在每个 block 的前面。

因为这个 FOR 的编码是有解压缩成本的。利用 skip list，除了跳过了遍历的成本，也跳过了解压缩这些压缩过的 block 的过程，从而节省了 cpu。





































