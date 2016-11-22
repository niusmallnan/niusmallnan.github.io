=======================================
屌丝级的cinder&swift的解决方案
=======================================
本文展示在没有任何存储设备的时候，如何让cinder和swift的服务可用，当然这要用到linux的loop device的特性，就是用大文件块来模拟磁盘。



cinder服务可用
=================
在没有多余磁盘时，cinder-api、cinder-schedule是可用的，唯独cinder-volume，所以我们要让cinder-volume在LVMISCSIDriver驱动下可用。

假设我们在root根目录下操作，cinder-volumes文件块的路径是 /root/cinder-volumes

安装cinder::

    $ apt-get install cinder-api cinder-scheduler 
    $ apt-get install cinder-volume lvm2


准备磁盘(50G)::
    

    $ dd if=/dev/zero of=cinder-volumes bs=1 count=0 seek=50G 
    $ losetup /dev/loop2 cinder-volumes
    $ fdisk /dev/loop2
        Type in the followings:
            n 
            p
            1
            ENTER 
            ENTER 
            t
            8e
            w
    
    $ partprobe

创建物理卷组::

    pvcreate /dev/loop2
    vgcreate cinder-volumes /dev/loop2


为防重启服务器后丢失::

    在rc.local添加
    losetup /dev/loop2 /root/cinder-volumes


修改LVM filter::
    
    # /etc/lvm/lvm.conf
    filter = [ "a/sda1/", "a/loop2/", "r/.*/"]

    

swift服务可用
=================



