# redis 数据结构
https://mp.weixin.qq.com/s/MGcOl1kGuKdA7om0Ahz5IA

![image](https://user-images.githubusercontent.com/42630862/146923710-799c1aac-4dcc-4ae7-902d-c29571bd6ecc.png)


![image](https://user-images.githubusercontent.com/42630862/146705535-3c742c9a-149d-499a-b267-6ebd2c60a7c0.png)


## 1 string
set   <key><value>添加键值对
 
*NX：当数据库中key不存在时，可以将key-value添加数据库
*XX：当数据库中key存在时，可以将key-value添加数据库，与NX参数互斥
*EX：key的超时秒数
*PX：key的超时毫秒数，与EX互斥

get   <key>查询对应键值
append  <key><value>将给定的<value> 追加到原值的末尾
strlen  <key>获得值的长度
setnx  <key><value>只有在 key 不存在时    设置 key 的值

incr  <key>
将 key 中储存的数字值增1
只能对数字值操作，如果为空，新增值为1
decr  <key>
将 key 中储存的数字值减1
只能对数字值操作，如果为空，新增值为-1
incrby / decrby  <key><步长>将 key 中储存的数字值增减。自定义步长。

### 数据结构
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

lpush/rpush  <key><value1><value2><value3> .... 从左边/右边插入一个或多个值。
lpop/rpop  <key>从左边/右边吐出一个值。值在键在，值光键亡。

rpoplpush  <key1><key2>从<key1>列表右边吐出一个值，插到<key2>列表左边。

lrange <key><start><stop>
按照索引下标获得元素(从左到右)
lrange mylist 0 -1   0左边第一个，-1右边第一个，（0-1表示获取所有）
lindex <key><index>按照索引下标获得元素(从左到右)
llen <key>获得列表长度 

linsert <key>  before <value><newvalue>在<value>的后面插入<newvalue>插入值
lrem <key><n><value>从左边删除n个value(从左到右)
lset<key><index><value>将列表key下标为index的值替换成value

### data structure

**3.0之前**
数据量小 压缩列表
数据量大 双向链表

### 原因

链表的缺陷也是有的，链表每个节点之间的内存都是不连续的，意味着无法很好利用 CPU 缓存。

能很好利用 CPU 缓存的数据结构就是数组，因为数组的内存是连续的，这样就可以充分利用 CPU 缓存来加速访问。
 
因此，Redis 的 list 数据类型在数据量比较少的情况下，会采用「压缩列表」作为底层数据结构的实现，压缩列表就是由数组实现的，下面我们会细说压缩列表。

### 压缩列表
压缩列表是 Redis 数据类型为 list 和 hash 的底层实现之一

当一个列表键（list）只包含少量的列表项，并且每个列表项都是小整数值，或者长度比较短的字符串，那么 Redis 就会使用压缩列表作为列表键（list）的底层实现。

当一个哈希键（hash）只包
 含少量键值对，并且每个键值对的键和值都是小整数值，或者长度比较短的字符串，那么 Redis 就会使用压缩列表作为哈希键（hash）的底层实现。

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
  
**3.2之后**
## quicklist

```
 typedef struct quicklist {
    //quicklist的链表头
    quicklistNode *head;      //quicklist的链表头
    //quicklist的链表头
    quicklistNode *tail; 
    //所有压缩列表中的总元素个数
    unsigned long count;
    //quicklistNodes的个数
    unsigned long len;       
    ...
} quicklist;
 
 
 typedef struct quicklistNode {
    //前一个quicklistNode
    struct quicklistNode *prev;     //前一个quicklistNode
    //下一个quicklistNode
    struct quicklistNode *next;     //后一个quicklistNode
    //quicklistNode指向的压缩列表
    unsigned char *zl;              
    //压缩列表的的字节大小
    unsigned int sz;                
    //压缩列表的元素个数
    unsigned int count : 16;        //ziplist中的元素个数 
    ....
} quicklistNode;
 
```
![image](https://user-images.githubusercontent.com/42630862/146890662-2b79f078-d174-41c7-bf4c-fba77a7d6a99.png)

 
 

## 3 Set

sadd <key><value1><value2> ..... 
将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略

   smembers <key>取出该集合的所有值。

    sismember <key><value>判断集合<key>是否为含有该<value>值，有1，没有0

    scard<key>返回该集合的元素个数。

    srem <key><value1><value2> .... 删除集合中的某个元素。
    
spop <key>随机从该集合中吐出一个值。
    
srandmember <key><n>随机从该集合中取出n个值。不会从集合中删除 。
    
smove <source><destination>value把集合中一个值从一个集合移动到另一个集合
    
sinter <key1><key2>返回两个集合的交集元素。
    
sunion <key1><key2>返回两个集合的并集元素。
    
sdiff <key1><key2>返回两个集合的差集元素(key1中的，不包含key2中的)

### data structure
### 哈希表，所有value指向同一个内部值
Set数据结构是dict字典，字典是用哈希表实现的。
Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

## 整数集合

```
typedef struct intset {
    //编码方式
    uint32_t encoding;
    //集合包含的元素数量
    uint32_t length;
    //保存元素的数组
    int8_t contents[];
} intset;
```
可以看到，保存元素的容器是一个 contents 数组，虽然 contents 被声明为 int8_t 类型的数组，但是实际上 contents 数组并不保存任何 int8_t 类型的元素，contents 数组的真正类型取决于 intset 结构体里的 encoding 属性的值。比如：

如果 encoding 属性值为 INTSET_ENC_INT16，那么 contents 就是一个 int16_t 类型的数组，数组中每一个元素的类型都是 int16_t；

如果 encoding 属性值为 INTSET_ENC_INT32，那么 contents 就是一个 int32_t 类型的数组，数组中每一个元素的类型都是 int32_t；

如果 encoding 属性值为 INTSET_ENC_INT64，那么 contents 就是一个 int64_t 类型的数组，数组中每一个元素的类型都是 int64_t；

不同类型的 contents 数组，意味着数组的大小也会不同。

### 整数集合的升级操作
整数集合会有一个升级规则，就是当我们将一个新元素加入到整数集合里面，如果新元素的类型（int32_t）比整数集合现有所有元素的类型（int16_t）都要长时，整数集合需要先进行升级，也就是按新元素的类型（int32_t）扩展 contents 数组的空间大小，然后才能将新元素加入到整数集合里，当然升级的过程中，也要维持整数集合的有序性。

整数集合升级的过程不会重新分配一个新类型的数组，而是在原本的数组上扩展空间，然后在将每个元素按间隔类型大小分割，如果 encoding 属性值为 INTSET_ENC_INT16，则每个元素的间隔就是 16 位。


## 4 Hash


hset <key><field><value>给<key>集合中的  <field>键赋值<value>

    hget <key1><field>从<key1>集合<field>取出 value 

    hmset <key1><field1><value1><field2><value2>... 批量设置hash的值

    hexists<key1><field>查看哈希表 key 中，给定域 field 是否存在。 

    hkeys <key>列出该hash集合的所有field

    hvals <key>列出该hash集合的所有value

    hincrby <key><field><increment>为哈希表 key 中的域 field 的值加上增量 1   -1

    hsetnx <key><field><value>将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .

    ### data structure
## data strcuture
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

zadd  <key><score1><value1><score2><value2>…
将一个或多个 member 元素及其 score 值加入到有序集 key 当中。

    zrange <key><start><stop>  [WITHSCORES]   
返回有序集 key 中，下标在<start><stop>之间的元素
带WITHSCORES，可以让分数一起和值返回到结果集。

    zrangebyscore key minmax [withscores] [limit offset count]
返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 

    zrevrangebyscore key maxmin [withscores] [limit offset count]               
同上，改为从大到小排列。 

    zincrby <key><increment><value>      为元素的score加上增量

    zrem  <key><value>删除该集合下，指定值的元素 

    zcount <key><min><max>统计该集合，分数区间内的元素个数 

    zrank <key><value>返回该值在集合中的排名，从0开始。

## data structure
    
zset底层使用了两个数据结构
（1）hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。
（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。


```
typedef struct zset {
    dict *dict;
    zskiplist *zsl;
} zset;
```
Zset 对象能支持范围查询（如 ZRANGEBYSCORE 操作），这是因为它的数据结构设计采用了跳表，而又能以常数复杂度获取元素权重（如 ZSCORE 操作），这是因为它同时采用了哈希表进行索引。

### 跳表：

https://zhuanlan.zhihu.com/p/109946103
![image](https://user-images.githubusercontent.com/42630862/146865893-fbd400ab-1ab8-41a8-82ac-5be95a6d8699.png)

```
typedef struct zskiplistNode {
    //Zset 对象的元素值
    sds ele;
    //元素权重值
    double score;
    //后向指针
    struct zskiplistNode *backward;

    //节点的level数组，保存每层上的前向指针和跨度
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span;
    } level[];
} zskiplistNode;
```


```
typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
```

Zset 对象要同时保存元素和元素的权重，对应到跳表节点结构里就是 sds 类型的 ele 变量和 double 类型的 score 变量。每个跳表节点都有一个后向指针，指向前一个节点，目的是为了方便从跳表的尾节点开始访问节点，这样倒序查找时很方便。

 随机层数
对于每一个新插入的节点，都需要调用一个随机算法给它分配一个合理的层数，源码在 t_zset.c/zslRandomLevel(void) 中被定义：
```
int zslRandomLevel(void) {
    int level = 1;
    while ((random()&0xFFFF) < (ZSKIPLIST_P * 0xFFFF))
        level += 1;
    return (level<ZSKIPLIST_MAXLEVEL) ? level : ZSKIPLIST_MAXLEVEL;
}
 ```
直观上期望的目标是 50% 的概率被分配到 Level 1，25% 的概率被分配到 Level 2，12.5% 的概率被分配到 Level 3，以此类推...有 2-63 的概率被分配到最顶层，因为这里每一层的晋升率都是 50%。

Redis 跳跃表默认允许最大的层数是 32，被源码中 ZSKIPLIST_MAXLEVEL 定义，当 Level[0] 有 264 个元素时，才能达到 32 层，所以定义 32 完全够用了。

 
 
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
 
https://mp.weixin.qq.com/s/CBfQEzEfjpbHocsg5QYBzQ
 
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
 
**4.0之后** •volatile-lfu：在设置了过期时间的key中使用LFU算法淘汰key•allkeys-lfu：在所有的key中使用LFU算法淘汰数据
 
 LRU(Least Recently Used)，即最近最少使用，是一种缓存置换算法。在使用内存作为缓存的时候，缓存的大小一般是固定的。当缓存被占满，这个时候继续往缓存里面添加数据，就需要淘汰一部分老的数据，释放内存空间用来存储新的数据。这个时候就可以使用LRU算法了。其核心思想是：如果一个数据在最近一段时间没有被用到，那么将来被使用到的可能性也很小，所以就可以被淘汰掉。
 
 LFU算法是Redis4.0里面新加的一种淘汰策略。它的全称是Least Frequently Used，它的核心思想是根据key的最近被访问的频率进行淘汰，很少被访问的优先被淘汰，被访问的多的则被留下来。LFU算法能更好的表示一个key被访问的热度。假如你使用的是LRU算法，一个key很久没有被访问到，只刚刚是偶尔被访问了一次，那么它就被认为是热点数据，不会被淘汰，而有些key将来是很有可能被访问到的则被淘汰了。如果使用LFU算法则不会出现这种情况，因为使用一次并不会使一个key成为热点数据。LFU一共有两种策略：


 
4.6.4.	maxmemory-samples

	设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，redis默认会检查这么多个key并选择其中LRU的那个。

	一般设置3到7的数字，数值越小样本越不准确，但性能消耗越小。

 # redis IO
 
 解决IO问题的最佳途径，使用集群版redis，使用集群 proxy来解决，以下还是针对于单节点
  
 1. 4.0 之前 单线程
 
 2. 4.0 之后lazy free
 
 3. 6.0 多线程IO
 
 ## 1. 4.0 之前 单线程
 ![image](https://user-images.githubusercontent.com/42630862/147028940-8e1ed168-0768-41d3-91cd-44a5c1981570.png)

 ![image](https://user-images.githubusercontent.com/42630862/147028958-0f646937-a9e5-41b0-ae07-6255d25e3761.png)

 
 ## 2. 4.0 之后lazy free
 
 如上所知，Redis在处理客户端命令时是以单线程形式运行，而且处理速度很快，期间不会响应其他客户端请求，但若客户端向Redis发送一条耗时较长的命令，比如删除一个含有上百万对象的Set键，或者执行flushdb，flushall操作，Redis服务器需要回收大量的内存空间，导致服务器卡住好几秒，对负载较高的缓存系统而言将会是个灾难。为了解决这个问题，在Redis 4.0版本引入了Lazy Free，将慢操作异步化，这也是在事件处理上向多线程迈进了一步。

如作者在其博客中所述，要解决慢操作，可以采用渐进式处理，即增加一个时间事件，比如在删除一个具有上百万个对象的Set键时，每次只删除大键中的一部分数据，最终实现大键的删除。但是，该方案可能会导致回收速度赶不上创建速度，最终导致内存耗尽。因此，Redis最终实现上是将大键的删除操作异步化，采用非阻塞删除（对应命令UNLINK），大键的空间回收交由单独线程实现，主线程只做关系解除，可以快速返回，继续处理其他事件，避免服务器长时间阻塞。

以删除（DEL命令）为例，看看Redis是如何实现的，下面就是删除函数的入口，其中，lazyfree_lazy_user_del是是否修改DEL命令的默认行为，一旦开启，执行DEL时将会以UNLINK形式执行。
 
 ## 3. 6.0 多线程IO
 
 
![image](https://user-images.githubusercontent.com/42630862/147048543-c7f46a0f-d14e-425b-8437-dd60d4eaf363.png)

 ![image](https://user-images.githubusercontent.com/42630862/147054087-d3a81e97-4ea2-4432-8380-54c06b77426f.png)

 
 

