https://docs.openstack.org/nova/latest/admin/cpu-topologies.html

# numa

Cell：NUMA Node 的通名词，供 Libvirt API 使用

vCPU：虚拟机的 CPU，根据虚拟机 NUMA 拓扑的不同，一个虚拟机 CPU 可以是一个 socket、core 或 thread。

pCPU：宿主机的 CPU，根据宿主机 NUMA 拓扑的不同，一个物理机 CPU 同样可以是一个 socket、core 或 thread。

Siblings Thread：兄弟线程，即由同一个 Core 超线程出来的 Threads。

Host NUMA Topology：宿主机的 NUMA 拓扑。

Guest NUMA Topology：虚拟机的 NUMA 拓扑。

# NUMA Topology
现在的服务器基本都支持 NUMA 拓扑，上文已经提到过，主要驱动 NUMA 体系结构应用的因素是 NUMA 具有的高存储访问带宽、有效的 Cache 效率以及灵活 PCIe I/O 设备的布局设计。但由于 NUMA 跨节点远程内存访问不仅延时高、带宽低、消耗大，还可能需要处理数据一致性的问题。因此，虚拟机的 vCPU 和内存在 NUMA 节点上的错误布局，将会导宿主机资源的严重浪费，这将抹掉任何内存与 CPU 决策所带来的好处。
所以，**标准的策略是尽量将一个虚拟机完全局限在单个 NUMA 节点内。**

# 1. instance NUMA placement policies( numa 亲和)

