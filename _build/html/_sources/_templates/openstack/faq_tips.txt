=======================================
OpenStack问题解决大全
=======================================
整理一些工作中遇到的问题以及解决办法

Horizon cookie配置
==================
修改 /usr/share/openstack-dashboard/openstack_dashboard/settings.py 文件::

    SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'
    SESSION_COOKIE_HTTPONLY = True
    SESSION_EXPIRE_AT_BROWSER_CLOSE = False
    SESSION_COOKIE_SECURE = False
    #SESSION_TIMEOUT = 1800
    SESSION_COOKIE_AGE = 3600

这样cookie就能保存一段时间，而不必每次浏览器重开，都要重新输入用户密码

易失性存储文件系统
==================
修改 /etc/nova/nova.conf ::

    [DEFAULT]
    default_ephemeral_format = ext3 # ext4  xfs

这样就可以指定ephemeral disk的默认文件系统了


nova-network浮动ip   
==================
参照下面::

    分配地址池
    用法： nova-manage floating create <ip_range> [--pool <pool>] [--interface <interface>]
    实例： nova-manage floating create 192.168.6.96/28 –pool demo
    参数为IP范围，前面是网络号，后面是掩码，给一个在线计算子网的地址  后面是名字，任意。
    删除地址池
    nova-manage floating delete <ip_range>
    列出现有的地址
    nova-manage floating list
    为实例分配一个IP地址
    nova floating-ip-create <pool>
    会列出1个可用的IP，例如192.168.0.100，后面的地址池名称可选。
    然后关联到实例
    nova add-floating-ip instance 192.168.0.100
    从实例移除IP
    移除操作则相反，首先解除关联。
    nova remove-floating-ip instance 192.168.0.100
    然后移除
    nova floating-ip-delete 192.168.0.100


nova中hostname不一致问题
==============================
nova服务中每个service都有一个host属性，这个host可以通过nova.conf 中配置得来，也会自动获取。

另一个host是compute_nodes的hypervisor_hostname，这个是通过Hypervisor的api获取的，比如libvirt，则可以通过virsh hostname 看到。

这两个host属性必须要保持一致，如果因为某种原因导致不一致，则会对一些服务产生影响，尤其是vm migration。

另外neutron agent中的host 一定要与 nova service中的host 保持一致。


各组件软件包升级
==============================
如果使用ubuntu OS，当官方出了新的release版本，各个组件都有新的deb可以安装，如果一个个安装是费时，这里利用ubuntu的包依赖关系，我便可轻松搞定升级，
以升级计算节点的nova和neutron为例::

    apt-get -y -o Dpkg::Options::="--force-confold" install nova-common neutron-common
    restart nova-compute
    restart neutron-plugin-openvswitch-agent

其他组件只需要安装对应的common包，即可做到升级，force-confold很重要，这能保证原先的配置不会被覆盖掉。

Haproxy代理nova-novnc
==============================
需要在haproxy上添加::
    
    listen spice_cluster
    bind <Virtual IP>:6080
    balance  source
    option  tcpka
    option  tcplog
    server controller1 10.0.0.1:6080 check inter 2000 rise 2 fall 5
    server controller2 10.0.0.2:6080 check inter 2000 rise 2 fall 5

所有nova组件添加配置并安装依赖包::
    
    #file nova.conf
    [DEFAULT]
    memcached_servers=<memcached_host>:11211

    apt-get install python-memcache

    #重启nova所有服务

tcpdump使用方法
==============================
整理如下::
    
    #查看来自某虚机到vlan的数据包
    #eth5为data-network网卡
    #适用于 vlan网络类型 排错
    tcpdump -i eth5  -n -e ether host <vm mac address>  -vv

更多使用方式请看 http://www.workrobot.com/sysadmin/security/tcpdump_expressions.html


