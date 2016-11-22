=======================================
VPNaas部署及实现原理
=======================================
VPNaas是基于neutron组件实现的VPN服务，目前社区提供了基于openswan实现的IPsec VPN以及基于硬件实现CiscoCsrIPsec VPN。
IPsec VPN的原理可以参考这篇文章 `OpenStack IPSec VPNaaS <http://www.openstack.cn/p612.html>`_


部署方法
===================
首先在neutron-server上修改neutron.conf，添加一个service_plugins::
    
    #除了router外 还需添加vpnaas
    service_plugins = router,vpnaas

    ##重启neutron-server
    restart neutron-server


在IceHouse版本中，VPNaas没法通过软件包来安装，否则会把l3-agent remove掉，所以我们需要手动做一些工作。

在/usr/bin目录下添加服务入口neutron-vpnaas-agent，内容如下::

    #! /usr/bin/python
    # PBR Generated from u'console_scripts'

    import sys

    from neutron.services.vpn.agent import main


    if __name__ == "__main__":
        sys.exit(main())

添加可执行权限::
    
    chmod a+x  /usr/bin/neutron-vpnaas-agent


添加upstart自启动服务，新建/etc/init/neutron-vpnaas-agent.conf内容如下::

    # vim:set ft=upstart ts=2 et:
    description "Neutron VPN Agent"
    author "niusmallnan <zhangzb@neunn.com>"

    start on runlevel [2345]
    stop on runlevel [!2345]

    respawn

    chdir /var/run

    pre-start script
        mkdir -p /var/run/neutron
        chown neutron:root /var/run/neutron
        # Check to see if openvswitch plugin in use by checking
        # status of cleanup upstart configuration
        if status neutron-ovs-cleanup; then
            start wait-for-state WAIT_FOR=neutron-ovs-cleanup WAIT_STATE=running WAITER=neutron-vpnaas-agent
        fi
    end script

    exec start-stop-daemon --start --chuid neutron --exec /usr/bin/neutron-vpnaas-agent -- \
            --config-file=/etc/neutron/neutron.conf --config-file=/etc/neutron/vpn_agent.ini \
            --config-file=/etc/neutron/l3_agent.ini --log-file=/var/log/neutron/vpnaas-agent.log


安装openswan，用于实现IPsec VPN::
    
    apt-get install openswan

修改horizon的local_setting, enable_lb为True::
    
    OPENSTACK_NEUTRON_NETWORK = {
        'enable_lb': True,
        'enable_firewall': True,
        'enable_quotas': True,
        'enable_vpn': True,
        'profile_support': None,
        'profile_support': 'cisco',
    }


启动vpnaas::

    start neutron-vpnaas-agent


使用注意事项
===================
如果vm之间的对端网络无法访问，则需要重启两端的neutron-vpnaas-agent服务。

添加IPsec站点连接时，伙伴网的网关填写的是对端网络external-net的router-interface地址。







