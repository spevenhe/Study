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



## 7 bitmaps



## 8 geospatial




