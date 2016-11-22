=======================================
Nova Compute集成iSCSI
=======================================
注意下面操作全部在root下执行

首先挂载在物理机上挂载LUN::

    #发现
    iscsiadm --mode discovery --type sendtargets --portal 172.26.0.1
    172.26.0.1:3260,257 iqn.2006-08.com.huawei:oceanstor:210004f938ebfdb4::20100:172.26.0.1

    #登录
    iscsiadm -m node -T iqn.2006-08.com.huawei:oceanstor:210004f938ebfdb4::20100:172.26.0.1 -l

    #通过 fdisk -l 检查是否挂载成功
    Disk /dev/sdb: 128.8 GB, 128849018880 bytes
    255 heads, 63 sectors/track, 15665 cylinders, total 251658240 sectors

    #设置该iSCSI session开机自动挂载
    iscsiadm -m node -T iqn.2006-08.com.huawei:oceanstor:210004f938ebfdb4::20100:172.26.0.1  --op update -n node.startup -v automatic

挂载nova目录，以便虚拟机能够被创建在LUN中::
    
    # 格式化块设备
    mkfs.ext4 /dev/sdb

    # mount 目录
    mount /dev/sdb /var/lib/nova/instances/

    
    #权限
    chown -R nova:nova /var/lib/nova/instances/


设置开机自动mount::
    

    #取得设备的uuid
    ls -l /dev/disk/by-uuid

    #修改 /etc/fstab  添加如下（UUID对应的值替换掉）
    UUID=722360e9-98d0-4a6d-b0d7-b2336e51847  /var/lib/nova/instances ext4 defaults,nobootwait 0 0
    
    #修改rc.local 在exit 0前添加
    mount -a

重启nova服务::

    restart nova-compute

如需深度校验，可以重启机器验证下，是否能够自动挂载::

    root@ncloud-compute-19:~# df -hl
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda1       1.1T  2.0G  1.1T   1% /
    none            4.0K     0  4.0K   0% /sys/fs/cgroup
    udev             63G  4.0K   63G   1% /dev
    tmpfs            13G 1000K   13G   1% /run
    none            5.0M     0  5.0M   0% /run/lock
    none             63G     0   63G   0% /run/shm
    none            100M     0  100M   0% /run/user
    /dev/sdb        118G  1.1G  111G   1% /var/lib/nova/instances








