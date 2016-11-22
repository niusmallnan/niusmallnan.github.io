=======================================
KVM虚拟机IO性能调优
=======================================
我们知道OpenStack中nova组件能够支持很多种虚拟化方式，官方目前主推的KVM，面向KVM虚拟化支持的功能也是最多的，它能够和neutron组件有很好的结合。本文的重点主要讨论KVM虚拟机IO性能调优，但是需要提前说明的是不同的场景需要选用不同的策略，这里没有银弹。


因为我不是专门做虚拟化技术的，所以本文只是把nova中的调优方法总结一下，至于哪个好，自己试！

aio 调度策略
=====================
aio有两种调度策略，nativa 和 threads，nova中并没有激活使用aio，可能是考虑到除了aio还有bio以及nio可以使用的缘故，这些应该交给管理员去配置。

nova中的代码没有指定aio，所以我们的简单修改下代码::

    #edit /usr/lib/python2.7/dist-packages/nova/virt/libvirt/config.py

    self.disk_total_iops_sec = None
    self.io = "native"   #add

    def format_dom(self):
        dev = super(LibvirtConfigGuestDisk, self).format_dom()

        dev.set("type", self.source_type)
        dev.set("device", self.source_device)
        if (self.driver_name is not None or
            self.driver_format is not None or
            self.driver_cache is not None):
            drv = etree.Element("driver")
            if self.driver_name is not None:
                drv.set("name", self.driver_name)
            if self.driver_format is not None:
                drv.set("type", self.driver_format)
            if self.driver_cache is not None:
                drv.set("cache", self.driver_cache)
            if self.io is not None:  #add
                drv.set("io", self.io)   #add
            dev.append(drv)

重启nova-compute服务，已经创建的虚拟机硬重启一下，就能支持aio了，查看qemu-kvm的进程，可以看到aio的参数。

目前没有配置项可以设置aio，社区里有个BP是关于这个问题的，https://blueprints.launchpad.net/nova/+spec/improve-nova-kvm-io-support ，还没有被实现，core成员给的回复是io策略比较多，nova不适合做这些事，这些应该交给管理员去配置kvm来实现。

那么这个aio该如何使用呢？有个普遍的认识是这样的，io=native for block device based VMs，io=threads for file-based VMs。就是说qcow2的用thread，基于块设备创建vm就用nativa，但这并不是绝对的，最好在你的场景下都测试一下，选择最优的。

io cache策略
=====================
激活了aio后，vm的io性能能够比较接近物理机，但是如果你对性能有更高的追求，仍然有办法，就是开启io cache，让物理内存为你的vm io操作提速。

查看qemu-kvm的进程，能看到 cache=none 的参数，这是nova默认的，默认是关闭cache，如果开启cache，则需要做一些配置::

    #edit /etc/nova/nova-compute.conf

    [libvirt]
    disk_cachemodes = file=default, block=none
    
file是针对file-backend的disk，block是针对块设备的disk，这个你可以通过vm的 libvir.xml 对比看一下就了解。default 和 none 是指cache 模式，nova提供很多种cache mode，通过源码可知::

    # nova/virt/libvirt/driver.py 

    self.disk_cachemodes = {}
    self.valid_cachemodes = ["default", 
                        "none", 
                        "writethrough", 
                        "writeback",
                        "directsync",
                        "unsafe",
                        ]

指定了cache之后，vm的io性能将会有质的飞跃，但是cache写满后，性能就回到普通状态，cache刷到硬盘后，io性能又会提升。

block or file backend 
=====================
如果前面所说file backend就是qcow2这个模式，qcow2虽然有很多先进的特性，但是io性能上一直都是有问题的，而基于块设备block backend创建的vm，io性能上是最佳的。

实现block backend可以有两种方法

- 基于cinder volume创建vm，这时候宿主机通过iSCSI挂载远端volume，vm可将其当做本地volume使用。

- 基于本地lvm创建vm，需要提前设置好本地的volume group。

通过本地lvm方式创建vm，需要进行如下配置::

    确认lvm2有木有安装，没有则装之

    修改/etc/nova/nova-compute.conf
    [libvirt]
    images_type=lvm
    images_volume_group=nova-volumes

    系统需要有一块裸设备(比如 /dev/sdb)，执行下面命令:
    pvcreate /dev/sdb
    vgcreate nova-volumes /dev/sdb

    重启nova-compute服务

block backend 的性能无疑是最好的，但是其他特性很受限制，比如快照，block backend 只能使用raw镜像，快照会很大，非常不方便。

按照自己的场景，各取所需吧。


