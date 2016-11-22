.. niusmallnan documentation master file, created by
   sphinx-quickstart on Tue Feb 18 13:49:43 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


=======================================
Arista SDN部署
=======================================
Arista的SDN方案是基于Ml2插件，Arista的mechanism_driver也已经集成到neutron的源码中，本文只涉及openstack的配置如何调整，对于arista交换机如何设置请咨询arista的相关人员。


部署环境
======================
我们有一个简单的部署环境，但不是All-In-One这种，测试All-In-One对实际生产环境，意义不大，所以我们必须有一个简单的节点分离的部署环境。

在部署arista方案之前，你必须把H版neutron之类的都配好了，比如你已经实现了ovs plugin的方案，如果还没有，那么您还是从基础一点一点来吧。

我们这里有::

    ncloud-controller   11.11.11.101
    ncloud-compute-a1   11.11.11.103
    ncloud-compute-a2   11.11.11.104
    ncloud-network      11.11.11.105

11.11.11.0/24是我们的management-network，其它还应该有个data-network，不过这个部分和本文内容无太大关系，因为组件之间的通信都是走management-network的。

安装步骤
======================
我们这里测试的是vlan模式，你如果想要测试VXLAN或者GRE之类的，建议你提前向厂商声明，因为支持vlan和VXLAN的交换机是不一样的。

清空并初始化
--------------
如果你之前neutron中存在network、port、router之类的，那么你必须全部删除掉，对应的虚拟机也一并删除。

每个节点ovs和namespace的信息也要全部清空，在所有的节点都执行如下命令::

    neutron-ovs-cleanup
    neutron-netns-cleanup


停止controller节点上neutron-server的服务，并drop掉neutron对应的database。

然后我们要重建ovs的bridge(如果之前有请删除)，我的eth2网卡是data-network，eth0可以走通外网，那么ovs的操作如下:

compute节点::
    
    ovs-vsctl add-br br-int
    ovs-vsctl add-br br-eth2
    ovs-vsctl add-port br-eth2 eth2


network节点::
    
    ovs-vsctl add-br br-int
    ovs-vsctl add-br br-eth2
    ovs-vsctl add-port br-eth2 eth2
    
    #external network bridge
    ovs-vsctl add-br br-ex
    ovs-vsctl add-port br-ex eth0 


各节点配置
--------------
首先在所有节点上安装ml2 plugin，因为ml2插件的源码已经集成的neutron中，所以你只需把配置文件拷到对应的目录，就可以激活ml2插件了。

ml2的配置各个节点都需要一份，ml2插件的配置内容如下::
    
    ml2/ml2_conf.ini
    tenant_network_types = vlan
    mechanism_drivers = openvswitch,arista
    network_vlan_ranges = default:1000:2999

    openvswitch/ovs_neutron_plugin.ini
    bridge_mappings = default:br-eth2

    neutron.conf
    core_plugin = neutron.plugins.ml2.plugin.Ml2Plugin
    service_plugins = neutron.services.l3_router.l3_router_plugin.L3RouterPlugin

    
创建数据库
-------------
创建neutron对应的database，然后在controller节点执行如下操作::

    neutron-db-manage --config-file /etc/neutron/neutron.conf \
        --config-file /etc/neutron/plugins/ml2/ml2_conf_arista.ini \
        --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head


启动服务
-------------
neutron-server和ovs-agent服务启动有些变化，请参考下面。

contro1ler::

    /usr/bin/python /usr/bin/neutron-server \
        --config-file /etc/neutron/neutron.conf \
        --config-file /etc/neutron/plugins/ml2/ml2_conf_arista.ini \
        --config-file /etc/neutron/plugins/ml2/ml2_conf.ini \
        --log-file /var/log/neutron/server.log


compute && network::
    
    /usr/bin/python /usr/bin/neutron-openvswitch-agent \
        --config-file=/etc/neutron/neutron.conf \
        --config-file=/etc/neutron/plugins/ml2/ml2_conf.ini \
        --config-file=/etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini \
        --log-file=/var/log/neutron/openvswitch-agent.log


其余neutron服务正常启动即可，
    

其它参考
======================
http://www.revolutionlabs.net/2013/11/part-2-how-to-install-openstack-havana_15.html

或者邮件我zhangzhibo521@gmail.com


