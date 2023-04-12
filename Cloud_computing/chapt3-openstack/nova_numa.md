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

## 默认：实例跨numa 分布，numa awareness 必须显示指定 启用 NUMATopologyFilter

```
# nova.conf
[DEFAULT]
...
scheduler_default_filters=...,NUMATopologyFilter
packing_host_numa_cells_allocation_strategy=pack/spread
```

pack 跟spread 还会影响 pcie 设备

Nova 的 NUMA 亲和原则是：将 Guest vCPU/Mem 都分配在同一个 NUMA Node 上，充分使用 NUMA node local memory，避免跨 Node 访问 remote memory。
```
openstack flavor set FLAVOR-NAME \
    --property hw:numa_nodes=FLAVOR-NODES \
    --property hw:numa_cpus.N=FLAVOR-CORES \
    --property hw:numa_mem.N=FLAVOR-MEMORY
 ```
FLAVOR-NODES（整数）：设定 Guest NUMA nodes 的个数。如果不指定，则 Guest vCPUs 可以在任意可用的 Host NUMA nodes 上浮动。

N：整数，Guest NUMA nodes ID，取值范围在 [0, FLAVOR-NODES-1]。

FLAVOR-CORES（逗号分隔的整数）：设定分配到 Guest NUMA node N 上运行的 vCPUs 列表。如果不指定，vCPUs 在 Guest NUMA nodes 之间平均分配。

FLAVOR-MEMORY（整数）：单位 MB，设定分配到 Guest NUMA node N 上 Memory Size。如果不指定，Memory 在 Guest NUMA nodes 之间平均分配。

举例：Flavor定义Guest有4个vCPU，4096MB内存，设定Guest的NUMA Topology为2个NUMA Node，vCPU 0、1运行在 NUMA Node 0上，vCPU 2、3运行在NUMA Node 1上。并且占用NUMA Node 0的Memory 2048MB，占用 NUMA Node 1 的Memory 2048MB。

openstack flavor set aze-FLAVOR \ 
    --property hw:numa_nodes=2 \ 
    --property hw:numa_cpus.0=0,1 \ 
    --property hw:numa_cpus.1=2,3 \ 
    --property hw:numa_mem.0=2048 \ 
    --property hw:numa_mem.1=2048

# 2. instance CPU pinning policies (CPU绑定)

## The functionality described below is currently only supported by the libvirt/KVM driver and requires some host configuration for this to work. Hyper-V does not support CPU pinning.

## 默认虚拟机的vcpu 不会分配到特定的宿主机cpu 上

只有对于需要  real-time or near real-time 场景，需要绑定cpu

 hw:cpu_policy extra spec and equivalent image metadata property. There are three policies: dedicated, mixed and shared (the default). The dedicated CPU policy is used to specify that all CPUs of an instance should use pinned CPUs. To configure a flavor to use the dedicated CPU policy, run:
```
$ openstack flavor set $FLAVOR --property hw:cpu_policy=dedicated
```
This works by ensuring PCPU allocations are used instead of VCPU allocations. As such, it is also possible to request this resource type explicitly. To configure this, run:
```
$ openstack flavor set $FLAVOR --property resources:PCPU=N
```
(where N is the number of vCPUs defined in the flavor).

Flavor-based policies take precedence over image-based policies. For example, if a flavor specifies a CPU policy of dedicated then that policy will be used. If the flavor specifies a CPU policy of shared and the image specifies no policy or a policy of shared then the shared policy will be used. However, the flavor specifies a CPU policy of shared and the image specifies a policy of dedicated, or vice versa, an exception will be raised. This is by design. Image metadata is often configurable by non-admin users, while flavors are only configurable by admins. By setting a shared policy through flavor extra-specs, administrators can prevent users configuring CPU policies in images and impacting resource utilization.

# 3. instance CPU thread pinning policies (CPU线程绑定)

针对于 SMT（Simultaneous Multithreading，同步多线程）， **SMT 中 cpu 被称为 Siblings**
![image](https://user-images.githubusercontent.com/42630862/231388725-1e523a5f-7826-44b3-bc3f-446e6415b3c2.png)

# 4. instance emulator thread pinning policies (模拟器（underlying hypervisor）CPU线程绑定)
![image](https://user-images.githubusercontent.com/42630862/231391503-f8fca239-0990-4805-941a-530ff5b5dfd5.png)

# 5. Customizing instance CPU topologies

![image](https://user-images.githubusercontent.com/42630862/231392671-09f7dbf6-285b-4948-8905-790c924303fd.png)


# 6 Configuring libvirt compute nodes for CPU pinning
nova 对于宿主机的cpu 有两种管理方式，对于没有绑定的，使用vcpu， 并且可以过量分配，对于绑定的cpu， 使用pcpu。
![image](https://user-images.githubusercontent.com/42630862/231394126-1815fe07-84fa-490e-931b-07dd098e0d31.png)

# 7 Configuring CPU power management for dedicated cores


