# redis 数据结构

![image](https://user-images.githubusercontent.com/42630862/146705535-3c742c9a-149d-499a-b267-6ebd2c60a7c0.png)


## 1 string
### sds
![image](https://user-images.githubusercontent.com/42630862/146707090-b3f6b6d2-546e-41cb-b73b-4d5c9ae3c4fd.png)

结构中的每个成员变量分别介绍下：

len，SDS 所保存的字符串长度。这样获取字符串长度的时候，只需要返回这个变量值就行，时间复杂度只需要 O（1）。

alloc，分配给字符数组的空间长度。这样在修改字符串的时候，可以通过 alloc - len 计算 出剩余的空间大小，然后用来判断空间是否满足修改需求，如果不满足的话，就会自动将 SDS  的空间扩展至执行修改所需的大小，然后才执行实际的修改操作，所以使用 SDS 既不需要手动修改 SDS 的空间大小，也不会出现前面所说的缓冲区益处的问题。

flags，SDS 类型，用来表示不同类型的 SDS。一共设计了 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64，后面在说明区别之处。

buf[]，字节数组，用来保存实际数据。不需要用 “\0” 字符来标识字符串结尾了，而是直接将其作为二进制数据处理，可以用来保存图片等二进制数据。它即可以保存文本数据，也可以保存二进制数据，所以叫字节数组会更好点。

总的来说，Redis 的 SDS 结构在原本字符数组之上，增加了三个元数据：len、alloc、flags，用来解决 C 语言字符串的缺陷。

O（1）复杂度获取字符串长度
C 语言的字符串长度获取 strlen 函数，需要通过遍历的方式来统计字符串长度，时间复杂度是 O（N）。

而 Redis 的 SDS 结构因为加入了 len 成员变量，那么获取字符串长度的时候，直接返回这个变量的值就行，所以复杂度只有 O（1）。

二进制安全
因为 SDS 不需要用 “\0” 字符来标识字符串结尾了，而且 SDS 的 API 都是以处理二进制的方式来处理 SDS 存放在 buf[] 里的数据，程序不会对其中的数据做任何限制，数据写入的时候时什么样的，它被读取时就是什么样的。

通过使用二进制安全的 SDS，而不是 C 字符串，使得 Redis 不仅 可以保存文本数据，也可以保存任意格式的二进制数据。

不会发生缓冲区溢出
C 语言的字符串标准库提供的字符串操作函数，大多数（比如 strcat 追加字符串函数）都是不安全的，因为这些函数把缓冲区大小是否满足操作的工作交由开发者来保证，程序内部并不会判断缓冲区大小是否足够用，当发生了缓冲区溢出就有可能造成程序异常结束。

所以，Redis 的 SDS 结构里引入了 alloc 和 leb 成员变量，这样 SDS API 通过 alloc - len 计算，可以算出剩余可用的空间大小，这样在对字符串做修改操作的时候，就可以由程序内部判断缓冲区大小是否足够用。

而且，当判断出缓冲区大小不够用时，Redis 会自动将扩大 SDS 的空间大小，以满足修改所需的大小。

在扩展 SDS 空间之前，SDS API 会优先检查未使用空间是否足够，如果不够的话，API 不仅会为 SDS 分配修改所必须要的空间，还会给 SDS 分配额外的「未使用空间」。

这样的好处是，下次在操作 SDS 时，如果 SDS 空间够的话，API 就会直接使用「未使用空间」，而无须执行内存分配，有效的减少内存分配次数。

所以，使用 SDS 即不需要手动修改 SDS 的空间大小，也不会出现缓冲区溢出的问题。

节省内存空间
SDS 结构中有个 flags 成员变量，表示的是 SDS 类型。

Redos 一共设计了 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64。

这 5 种类型的主要区别就在于，它们数据结构中的 len 和 alloc 成员变量的数据类型不同，

## 2 List

数据量小 压缩列表
数据量大 双向快速链表

### 原因

链表的缺陷也是有的，链表每个节点之间的内存都是不连续的，意味着无法很好利用 CPU 缓存。

能很好利用 CPU 缓存的数据结构就是数组，因为数组的内存是连续的，这样就可以充分利用 CPU 缓存来加速访问。

因此，Redis 的 list 数据类型在数据量比较少的情况下，会采用「压缩列表」作为底层数据结构的实现，压缩列表就是由数组实现的，下面我们会细说压缩列表。

### 压缩列表
压缩列表是 Redis 数据类型为 list 和 hash 的底层实现之一

当一个列表键（list）只包含少量的列表项，并且每个列表项都是小整数值，或者长度比较短的字符串，那么 Redis 就会使用压缩列表作为列表键（list）的底层实现。

当一个哈希键（hash）只包含少量键值对，并且每个键值对的键和值都是小整数值，或者长度比较短的字符串，那么 Redis 就会使用压缩列表作为哈希键（hash）的底层实现。

压缩列表是 Redis 为了节约内存而开发的，它是由连续内存块组成的顺序型数据结构，有点类似于数组。

![image](https://user-images.githubusercontent.com/42630862/146708616-0fb923f3-5f0b-493d-a6bf-2227a2d881c9.png)


压缩列表在表头有三个字段：

zlbytes，记录整个压缩列表占用对内存字节数；

zltail，记录压缩列表「尾部」节点距离起始地址由多少字节，也就是列表尾的偏移量；

zllen，记录压缩列表包含的节点数量；

zlend，标记压缩列表的结束点，特殊值 OxFF（十进制255）。

压缩列表节点包含三部分内容：

prevlen，记录了前一个节点的长度；

encoding，记录了当前节点实际数据的类型以及长度；

data，记录了当前节点的实际数据；


### 快速链表 quicklist

![image](https://user-images.githubusercontent.com/42630862/145772592-21f78ac1-fa48-4283-a82c-31d33393dfd6.png)

```

typedef struct list {

    //链表头节点
    listNode *head;
    
    //链表尾节点
    listNode *tail;
    
    //节点值复制函数
    void *(*dup)(void *ptr);
    
    //节点值释放函数
    void (*free)(void *ptr);
    
    //节点值比较函数
    int (*match)(void *ptr, void *key);
    
    //链表节点数量
    unsigned long len;
} list;

```
![image](https://user-images.githubusercontent.com/42630862/146708087-08027409-4a03-441d-913f-f7d539a23a71.png)

压缩列表节点包含三部分内容：

prevlen，记录了前一个节点的长度；

encoding，记录了当前节点实际数据的类型以及长度；

data，记录了当前节点的实际数据；


## 3 Set

### 哈希表，所有value指向同一个内部值
Set数据结构是dict字典，字典是用哈希表实现的。
Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

## 4 Hash

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

![image](https://user-images.githubusercontent.com/42630862/146759394-094a6cd2-1039-4134-9d5a-e2a65302300a.png)

![image](https://user-images.githubusercontent.com/42630862/146759429-fd6530c3-977d-43fb-8800-cfea1bd260c8.png)
![image](https://user-images.githubusercontent.com/42630862/146759544-2170b67e-78bd-49cc-9b06-e5cb801fdfd3.png)

这个过程看起来简单，但是其实第二步很有问题，如果「哈希表 1 」的数据量非常大，那么在迁移至「哈希表 2 」的时候，因为会涉及大量的数据拷贝，此时可能会对 Redis 造成阻塞，无法服务其他请求。

渐进式 rehash
为了避免 rehash 在数据迁移过程中，因拷贝数据的耗时，影响 Redis 性能的情况，所以 Redis 采用了渐进式 rehash，也就是将数据的迁移的工作不再是一次性迁移完成，而是分多次迁移。

渐进式 rehash 步骤如下：

给「哈希表 2」 分配空间；

在 rehash 进行期间，每次哈希表元素进行新增、删除、查找或者更新操作时，Redis 除了会执行对应的操作之外，还会顺序将「哈希表 1 」中索引位置上的所有 key-value 迁移到「哈希表 2」 上；

随着处理客户端发起的哈希表操作请求数量越多，最终会把「哈希表 1 」的所有 key-value 迁移到「哈希表 2」，从而完成 rehash 操作。

这样就巧妙地把一次性大量数据迁移工作的开销，分摊到了多次处理请求的过程中，避免了一次性 rehash 的耗时操作。

在进行渐进式 rehash 的过程中，会有两个哈希表，所以在渐进式 rehash 进行期间，哈希表元素的删除、查找、更新等操作都会在这两个哈希表进行。

比如，查找一个 key 的值的话，先会在哈希表 1 里面进行查找，如果没找到，就会继续到哈希表 2 里面进行找到。

另外，在渐进式 rehash 进行期间，新增一个 key-value 时，会被保存到「哈希表 2 」里面，而「哈希表 1」 则不再进行任何添加操作，这样保证了「哈希表 1 」的 key-value 数量只会减少，随着 rehash 操作的完成，最终「哈希表 1 」就会变成空表。


## 5 zset
zset底层使用了两个数据结构
（1）hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。
（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。


## 6 hyperlog
HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。
在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

**具体参考 hyperlog**


## 7 bitmaps

Redis提供了Bitmaps这个“数据类型”可以实现对位的操作：

（1）	Bitmaps本身不是一种数据类型， 实际上它就是字符串（key-value） ， 但是它可以对字符串的位进行操作。

（2）	Bitmaps单独提供了一套命令， 所以在Redis中使用Bitmaps和使用字符串的方法不太相同。 可以把Bitmaps想象成一个以位为单位的数组， 数组的每个单元只能存储0和1， 数组的下标在Bitmaps中叫做偏移量。

![image](https://user-images.githubusercontent.com/42630862/146502673-a8e04c9c-a6d4-41be-94f6-fbb1d62d2718.png)


### 命令
1、setbit
（1）格式
setbit<key><offset><value>设置Bitmaps中某个偏移量的值（0或1）

2、getbit
（1）格式
getbit<key><offset>获取Bitmaps中某个偏移量的值

 3、bitcount
统计字符串被设置为1的bit数。一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。start 和 end 参数的设置，都可以使用负数值：比如 -1 表示最后一个位，而 -2 表示倒数第二个位，start、end 是指bit组的字节的下标数，二者皆包含。
（1）格式
bitcount<key>[start end] 统计字符串从start字节到end字节比特值为1的数量


## 8 geospatial
Redis 3.2 中增加了对GEO类型的支持。GEO，Geographic，地理信息的缩写。该类型，就是元素的2维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作。
 
 1、geoadd

（1）格式
geoadd<key>< longitude><latitude><member> [longitude latitude member...]   添加地理位置（经度，纬度，名称）

2 geopos  <key><member> [member...]  获得指定地区的坐标值
 
 3、geodist

（1）格式
geodist<key><member1><member2>  [m|km|ft|mi ]  获取两个位置之间的直线距离

 4、georadius
（1）格式
georadius<key>< longitude><latitude>radius  m|km|ft|mi   以给定的经纬度为中心，找出某一半径内的元素

 
 
# redis 内存

4.6.2.	maxmemory 

	建议必须设置，否则，将内存占满，造成服务器宕机

	设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。

	如果redis无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。

	但是对于无内存申请的指令，仍然会正常响应，比如GET等。如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的
是“不移除”的情况下，才不用考虑这个因素。
 
4.6.3.	maxmemory-policy

	volatile-lru：使用LRU算法移除key，只对设置了过期时间的键；（最近最少使用）

	allkeys-lru：在所有集合key中，使用LRU算法移除key

	volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键

	allkeys-random：在所有集合key中，移除随机的key

	volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key

	noeviction：不进行移除。针对写操作，只是返回错误信息
 
4.6.4.	maxmemory-samples

	设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，redis默认会检查这么多个key并选择其中LRU的那个。

	一般设置3到7的数字，数值越小样本越不准确，但性能消耗越小。



