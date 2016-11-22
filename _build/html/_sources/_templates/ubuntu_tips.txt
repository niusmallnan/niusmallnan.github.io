=======================================
Ubuntu问题及技巧整理
=======================================
整理一些工作中遇到的问题以及好的技巧


网卡识别乱序
=================
打开/etc/udev/rules.d/70-persistent-net.rules 文件，按照你想要的顺序重新编辑，然后重启机器。


iscsi挂载LUN
=================
主机内操作如下::

    发现
    iscsiadm --mode discovery --type sendtargets --portal 172.26.0.1
    172.26.0.1:3260,257 iqn.2006-08.com.huawei:oceanstor:210004f938ebfdb4::20100:172.26.0.1

    登录
    iscsiadm -m node -T iqn.2006-08.com.huawei:oceanstor:210004f938ebfdb4::20100:172.26.0.1 -l

    退出
    iscsiadm -m node -T iqn.2006-08.com.huawei:oceanstor:210004f938ebfdb4::20100:172.26.0.1 -u

    查看iscsi连接
    iscsiadm -m session

    删除登录痕迹
    iscsiadm -m node -T iqn.2011-09.com.example.ts1:disk1 -p 172.26.0.1:3260 -o delete

    查看客户端iqn
    /etc/iscsi/initiatorname.iscsi


网卡出现p1p1
=================
修改 /etc/default/grub ::
    
    GRUB_CMDLINE_LINUX_DEFAULT="biosdevname=0 quiet splash"
    GRUB_CMDLINE_LINUX="biosdevname=0"


修改 /etc/udev/rules.d/70-persistent-net.rules 把网卡映射信息配置好， 比如::

    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="04:f9:38:9a:7b:ec", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth1"

    # PCI device 0x14e4:0x1657 (tg3)
    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="04:f9:38:9a:7b:ee", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth3"

    # PCI device 0x14e4:0x1657 (tg3)
    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="04:f9:38:9a:7b:ed", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth2"

    # PCI device 0x14e4:0x1657 (tg3)
    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="04:f9:38:9a:7b:eb", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"

    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="ac:85:3d:9f:58:12", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth4"
    SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="ac:85:3d:9f:58:13", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth5"

执行update-grub

reboot重启机器后，即可修复。

内核优化参数
=================
内核优化参数::

    kernel.msgmnb = 65536
    # Controls the default maxmimum size of a mesage queue
    kernel.msgmax = 65536
    # Controls the maximum shared segment size, in bytes
    kernel.shmmax = 68719476736
    # Controls the maximum number of shared memory segments, in pages
    kernel.shmall = 4294967296
    kernel.shmmni = 4096
    kernel.sem = 50100 64128000 50100 1280
    fs.file-max = 7672460
    net.ipv4.ip_local_port_range = 9000 65000
    net.core.rmem_default = 1048576
    net.core.rmem_max = 4194304
    net.core.wmem_default = 262144
    net.core.wmem_max = 1048576
    net.ipv4.tcp_tw_recycle = 1
    net.ipv4.tcp_max_syn_backlog = 4096
    net.core.netdev_max_backlog = 10000
    vm.overcommit_memory = 0
    net.ipv4.ip_conntrack_max = 655360
    fs.aio-max-nr = 1048576
    net.ipv4.tcp_timestamps = 0
    
    
    /etc/security/limits.conf
    * soft    nofile  131072
    * hard    nofile  131072
    * soft    nproc   131072
    * hard    nproc   131072
    * soft    core    unlimited
    * hard    core    unlimited
    * soft    memlock 50000000
    * hard    memlock 50000000

swap分区设置
=================
内存小于4GB时，推荐不少于2GB的swap空间；

内存4GB~16GB，推荐不少于4GB的swap空间；

内存16GB~64GB，推荐不少于8GB的swap空间；

内存64GB~256GB，推荐不少于16GB的swap空间。

自动装载内核模块
=================
把要每次系统启动要自动装载的模块，写入/etc/modules，比如::
    
    $ cat /etc/modules
    acpiphp
    8021q





































