=======================================
宿主机资源隔离(未完)
=======================================
数据中心中少部分机器用于做控制节点，大部分机器都是需要运行虚拟化软件的，虚拟化平台的上有大量的vm，而宿主机本身的系统也会跑一些服务，那么这就势必会造成vm之间资源抢占，vm与宿主机系统之间的资源抢占，我们需要通过设定游戏规则，让他们在各自的界限内高效运行，减少冲突抢占。

nova配置相关
=================
在虚拟化资源使用上，我们可以通过nova来控制，OpenStack提供了一些配置，我们可以和容易的做到，修改nova-compute nova-conf::

    #虚拟机 vCPU 的绑定范围，可以防止虚拟机争抢宿主机进程的 CPU 资源
    #建议值是预留前几个物理 CPU
    vcpu_pin_set = 4-31

    #物理 CPU 超售比例，默认是 16 倍，超线程也算作一个物理 CPU
    cpu_allocation_ratio =  1

    #内存分配超售比例，默认是 1.5 倍，生产环境不建议开启超售
    ram_allocation_ratio = 1

    #内存预留量，这部分内存不能被虚拟机使用，以便保证系统的正常运行
    reserved_host_memory_mb = 16384 

    #磁盘预留空间，这部分空间不能被虚拟机使用，这里主要考虑给快照时产生的临时文件 预留空间
    #针对file backend的vm 需要预留
    #针对rdb backend的vm 不需要预留
    #针对lvm backend的vm 需要预留
    reserved_host_disk_mb = 40960


宿主机的配置
=================
我们可以让宿主机运行操作系统时候，更多的选择指定的几个核，这样就不会过多抢占虚拟化中虚机的vcpu调度，通过修改内核启动参数我们可以做到::

    #修改 /etc/default/grub 系统只使用前三个核 隔离其余核    
    GRUB_CMDLINE_LINUX_DEFAULT="isolcpus=4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31"

    #更新内核参数
    $ update-grub
    $ reboot

物理机磁盘空间分配，依据云环境的核心准则"隔离"，我们需要把系统运行所需的磁盘与虚拟化所需的磁盘分离开，通常我们会用raid分割出两块盘，一块盘/dev/sda挂载系统，另一块/dev/sdb 挂载虚拟化目录，特殊情况下需要把快照目录单独挂载一块硬盘，不同场景配置依据如下原则:

- 对于lvm backend，需要基于sdb创建volume group ，快照目录需独立挂载磁盘

- 对于file backend，sdb挂载虚拟机目录，快照目录迁出或者reserved_host_disk_mb预留一定空间

- 对于rbd backend，sdb挂载虚拟机目录，但是sdb的容量可以非常小即可


使用taskset进行CPU绑定，对于某些服务，如果我们只想让其在指定的CPU set中调度，那么可以通过下面的方式::

    #取得该服务的PID
    $ ps aux | grep xxx

    #查看CPU set
    $ taskset -pc <pid>

    #绑定CPU
    $ taskset -cp 1,2 <pid>


我们还可以用NUMA和Cgroup更加精细化地控制进程的资源使用，但是这些优化一般不需要，除非遇到性能问题你很清楚你在做什么，否则这种优化很可能导致性能不升反降，参考资料:

http://www.oracle.com/technetwork/cn/articles/servers-storage-admin/resource-controllers-linux-1506602-zhs.html


重要指标
=================
%st(Steal time) 是当 hypervisor 服务另一个虚拟处理器的时候，虚拟 CPU 等待实际 CPU 的时间的百分比。

Steal 值比较高的话，你需要向主机供应商申请扩容虚拟机。服务器上的另一个虚拟机可能拥有更大更多的 CPU 时间片，你可能需要申请升级以与之竞争。另外，高 steal 值可能意味着主机供应商在服务器上过量地出售虚拟机。如果升级了虚拟机， steal 值还是不降的话，你应该寻找另一家服务供应商。

低 steal 值意味着你的应用程序在目前的虚拟机上运作良好。因为你的虚拟机不会经常地为了 CPU 时间与其它虚拟机激烈竞争，你的虚拟机会更快地响应。这一点也暗示了，你的主机供应商没有过量地出售虚拟服务，绝对是一件好事情。

验证
=================
校验虚拟机vCPU亲和性，检查CPU是否隔离成功，创建一个虚机，然后通过libvirt查看::

    $ virsh list
     Id    Name                           State
     ----------------------------------------------------
      42    instance-00000553              running
      43    instance-00000596              running

    $ virsh  vcpuinfo 42
    VCPU:           0
    CPU:            4
    State:          running
    CPU time:       324210.2s
    CPU Affinity:   ----yyyyyyyyyyyyyyyyyyyyyyyyyyyy

这就说明，虚拟机的vCPU对物理CPU的前四个核非亲和，他一般就会在后面的物理CPU中调度执行各种服务。







