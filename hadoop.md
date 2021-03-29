# HadoopStudy
# HDFS 概述 

1）分布式

2）comodity hardware（low cost) 去IOE 
  
3）  fault-tolerance
  
4）  High throughput
  
 5） big data sets
 
 -----

HDFS是分布式文件系统

普通文件系统：Linux(ET2, ET4)， Windows(FAT, NTFS)，Mac。。。。  
		文件目录：C  /  
		存放的是文件或者文件夹  
		对外提供服务：创建 , 修改,删除，查看，移动  

普通文件系统 vs 分布式文件系统  
单价		vs 横跨n个机器  

# HDFS assumptions and goals
1） hadrware failure  
*	*blocksize = 128M  
*	*block 存在不同机器上， 3 copies  
2） streaming data access  

# HDFS

hdfs文件均存放在datanode上，namenode上不会存放文件。当客户上传一个文件后，namenode会先对文件作相应的处理（比如按照block大小进行分割）。这里主要讲述存放的一个整体过程以及如何快速的找到存放的节点位置信息。

实现namenode的源码中有一个与文件系统存储和管理有关的关键类FSNameSystem，里面有以下的一些概念：

INode: 用来存放文件及目录的基本信息：名称，父节点、修改时间，访问时间以及UGI信息。

INodeFile: 继承自INode，除INode信息外，还有组成这个文件的Blocks列表，重复因子，Block大小 

INodeDirectory：继承自INode，此外还有一个INode列表来组成文件或目录树结构 

Block(BlockInfo)：组成文件的物理存储，有BlockId，size ，以及时间戳

BlocksMap: 保存数据块到INode和DataNode的映射关系 

FSDirectory：保存文件树结构，HDFS整个文件系统是通过FSDirectory来管理 

FSImage：保存的是文件系统的目录树 

FSEditlog:  文件树上的操作日志 

FSNamesystem: HDFS文件系统管理 


