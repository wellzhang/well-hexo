---
title: kvm性能调优
categories: 201706
tags: 
- kvm
- libvirtd
---


## CPU Tuning
### Cache share tuning
对于物理 CPU，同一个 core 的 threads 共享 L2 Cache，同一个 socket 的 cores 共享 L3 cache，所以虚拟机的 vcpu 应当尽可能在同一个 core 和 同一个 socket 中，增加 cache 的命中率，从而提高性能。IBM 测试过，合理绑定 vcpu 能给 JVM 来的 16% 的性能提升[2]。

实现策略：虚拟机 vcpu 尽可能限定在一个 core 或者一个 socket 中。例如：当 vcpu 为 2 时，2 个 vcpu 应限定在同一个 core 中，当 vcpu 大于 2 小于 12 时，应限定在同一个 socket 中。

```xml
<vcpu placement='static' cpuset='0-5'>4</vcpu>       # cpuset 限定 vcpu
```

### NUMA tuning

网易运维团队测试得出：2 个 vcpu 分别绑定到不同 numa 节点的非超线程核上和分配到一对相邻的超线程核上的性能相差有 30%~40%(通过 SPEC CPU2006 工具测试)。可见，同一个虚拟机的 vcpu 需限定在同一个 NUMA 节点，并且分配该 NUMA 节点下的内存给虚拟机，保证虚拟机尽可能访问 local memory 而非 remote memory。

关于 memory allocation mode: If memory is overcommitted in strict mode and the guest does not have sufficient swap space, the kernel will kill some guest processes to retrieve additonal memory. Red Hat recommends using preferred allocation and specifying a single nodeset (for example, nodeset='0') to prevent this situation[1].

```xml
<numatune>

  <memory mode="preferred" nodeset="0"/> 

</numatune>
```

### IRQ tuning
CPU0 常用于处理中断请求，本身负荷较重[3]。事实上，线上环境 CPU0 完全处理 eth0 的 IRQ，eth1 的 IRQ 完全由另一个 CPU 处理，同时 CPU0 处理着大量的 CAL 类型中断。预留 2 个或者 4 个物理 CPU，这些 CPU 和对应网卡中断做绑定，使得其它 CPU 更好更完整的为云主机所用。  

### VCPU topology tuning
Selecting any desired number of sockets, but with only a single core and a single thread usually gives the best performance results[1]. 即 VCPU 的 topology 设置为 sockets = vcpu_number, cores = 1, threads = 1. 

##Disk IO Tuning
###Disk IO cache Tuning

kvm 支持多种虚拟机多种 IO Cache 方式：writeback, none, writethrough 等。性能上：writeback > none > writethrough，安全上 writeback < none < writethrough。For the best storage performance on guest operating systems that use raw disk volumes or partitions, completely avoid the page cache on the host[2].

```xml
<disk type='file' device='disk'>

  <driver name='qemu' type='qcow2' cache='none'/>  # cache 可为 writeback, none, writethrough，directsync，unsafe 等

  ...

</disk>
```


### Disk IO scheduler
cfq 参数调优。

##Memory Tuning
1. 关于 zone_reclaim_mode，设置为 disable 。
2. 关于 swappiness 参数，若 CPU macro-architecture 非 Intel Nehalem 架构，无需要修改优化。
3. 关闭 KVM 内存共享：When KSM merges across nodes on a NUMA host with multiple guest virtual machines, guests and CPUs from more distant nodes can suffer a significant increase of access latency to the merged KSM page[1].
  Disable KSM 方式有两种： 

1) 禁止某个 Guest 与其它 Guest 共享内存，XML 文件可配置为

``` xml

<memoryBacking>

    <nosharepages/>

</memoryBacking>
```


2)  禁止所有 Guest 直接共享内存，Host 配置为

```shell

echo 0 > /sys/kernel/mm/ksm/pages_shared

echo 0 > /sys/kernel/mm/ksm/pages_sharing
```

4. 打开透明大页：KVM guests can be deployed with huge page memory support in order to improve performance by increasing CPU cache hits against the Transaction Lookaside Buffer (TLB)[1].  
  打开透明大页方式有两种：

1) 允许某个 Guest 开启透明大页

Guest XML Format

```xml
<memoryBacking>

   <hugepages/>

</memoryBacking>
```

```shell

echo 25000 > /pro c/sys/vm/nr_hugepages

mount -t hugetlbfs hugetlbfs /dev/hugepages

service libvirtd restart
```


2) 允许 Host 中所有 Guest 开启透明大页

```shell
echo always > /sys/kernel/mm/transparent_hugepage/enabled

echo never > /sys/kernel/mm/transparent_hugepage/defrag

echo 0 > /sys/kernel/mm/transparent_hugepage/khugepaged/defrag
```


##Network IO Tuning
开启 vhost_net 模式[3]。 

参考资料

1.Red Hat Enterprise Linux 6 Virtualization Tuning Optimization Guide  
2.Tuning KVM for performance    IBM  
3.网易 openstack 部署运维实战


