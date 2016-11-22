=======================================
Nova实现虚拟机块迁移
=======================================
块迁移，是动态迁移（live migration）的一种，在不需要共享存储的情况下，将虚拟机迁移到其它宿主机上，迁移过程中不需要关闭虚拟机。拷贝instance文件到目标宿主机上，并重新加载网络策略。

如果虚机的根磁盘是raw格式的，那么整个迁移过程十分漫长，尤其是你的虚机如果构建在ceph的rbd上，那么使用这种方式迁移虚机是非常不明智的。而如果根磁盘是qcow的，那么想对会快速一些。

更多详情请参考 `官方文档 <http://docs.openstack.org/admin-guide-cloud/content/section_configuring-compute-migrations.html>`_

Nova的支持
======================
Nova迁移虚拟机是调用通过调用libvirt实现的，毕竟libvirt已经封装了针对各个虚拟化技术的统一接口，这点通过Nova代码可知::

    # Do live migration.
    try: 
        if block_migration:
            flaglist = CONF.block_migration_flag.split(',')
        else:
            flaglist = CONF.live_migration_flag.split(',')
        flagvals = [getattr(libvirt, x.strip()) for x in flaglist]
        logical_sum = reduce(lambda x, y: x | y, flagvals)

        dom = self._lookup_by_name(instance["name"])
        dom.migrateToURI(CONF.live_migration_uri % dest,
                        logical_sum,
                        None,
                        CONF.live_migration_bandwidth)
    

这里的libvirt是对底层libvirt接口封装的Python库，live_migration_uri是nova.conf的配置::
    
    
    # Migration target URI (any included "%s" is replaced with the
    # migration target hostname) (string value)
    live_migration_uri=qemu+tcp://%s/system


基本上Nova是不需要怎么配置的，真正的配置都在libvirt上。


libvirt配置
======================
主要是libvirtd.conf的配置，它通常在/etc/libvirt 下面，一般我们启用普通的TCP端口就行了，另外值得注意的是监听的ip地址一定要和该compute-node所在网络的ip一致::

    listen_tls = 0
    listen_tcp = 1
    tcp_port = "16509"
    listen_addr = "11.11.11.103"
    auth_tcp = "none" #如果你不想搞那些繁琐的鉴权配置

ubuntu下 /etc/default/libvirt-bin 也要修改::
    
    libvirtd_opts="-d -l"

    #重启libvirtd
    service libvirt-bin restart

然后你就可以迁移你的虚机了，注意使用命令行迁移的时候一定要加--block-migrate，否则Nova会判断是否在共享存储上。


