# redis 数据结构

## 1 string
### sds


## 2 List
### 快速链表 quicklist

![image](https://user-images.githubusercontent.com/42630862/145772592-21f78ac1-fa48-4283-a82c-31d33393dfd6.png)


## 3 Set

### 哈希表，所有value指向同一个内部值
Set数据结构是dict字典，字典是用哈希表实现的。
Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

## 4 Hash

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

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



