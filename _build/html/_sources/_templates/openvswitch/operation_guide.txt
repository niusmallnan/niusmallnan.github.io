=======================================
OVS操作指南
=======================================
Open vSwitch（下面简称为 OVS）是由 Nicira Networks 主导的，运行在虚拟化平台（例如 KVM，Xen）上的虚拟交换机。在虚拟化平台上，OVS 可以为动态变化的端点提供 2 层交换功能，很好的控制虚拟网络中的访问策略、网络隔离、流量监控等等。

OVS 遵循 Apache 2.0 许可证, 能同时支持多种标准的管理接口和协议。OVS 也提供了对 OpenFlow 协议的支持，用户可以使用任何支持 OpenFlow 协议的控制器对 OVS 进行远程管理控制。


在 OVS 中, 有几个非常重要的概念:

- Bridge: Bridge 代表一个以太网交换机（Switch），一个主机中可以创建一个或者多个Bridge 设备。

- Port: 端口与物理交换机的端口概念类似，每个 Port 都隶属于一个 Bridge。

- Interface: 连接到 Port 的网络接口设备。在通常情况下，Port 和 Interface 是一对一的关系, 只有在配置 Port 为 bond 模式后，Port 和 Interface 是一对多的关系

- Controller: OpenFlow 控制器。OVS 可以同时接受一个或者多个 OpenFlow 控制器的管理。

- datapath: 在 OVS 中，datapath 负责执行数据交换，也就是把从接收端口收到的数据包在流表中进行匹配，并执行匹配到的动作。

- Flow table: 每个 datapath 都和一个“flow table”关联，当 datapath 接收到数据之后， OVS 会在 flow table 中查找可以匹配的 flow，执行对应的操作, 例如转发数据到另外的端口。


注意本文内容均来自网络，个人整理，并非原创。

网桥管理
=================
非ovsdb数据库操作::

    #添加网桥
    ovs-vsctl add-br br-int

    #列出网桥
    ovs-vsctl list-br 

    #给网桥添加端口
    ovs-vsctl add-port br-int tap-xxx

    #列出挂载某网络接口的所有网桥
    ovs-vsctl port-to-br tap-xxx

    #查看全部信息
    ovs-vsctl show

ovsdb数据库操作::

    #通用格式为
    ovs-vsctl list/set/get/add/remove/clear/destroy table record column [value]

    #默认情况下ovsdb中包含的数据表
    bridge, controller,interface,mirror,netflow,open_vswitch,port,qos,queue,ssl,sflow

    #举例 查看所有网桥
    ovs-vsctl list bridge

    #举例 删除一条qos记录
    ovs-vsctl destroy qos <qos-id>

    #修改端口 p1 的 VLAN tag 为 101，使端口 p1 成为一个隶属于 VLAN 101 的端口
    ovs-vsctl set Port p1 tag=101

流规则管理
=================
每条流规则由一系列字段组成，分为基本字段、条件字段和动作字段三部分。

基本字段包括:

- 生效时间 duration_sec

- 所属表项 table_id

- 优先级 priority、

- 处理的数据包数 n_packets

- 空闲超时时间 idle_timeout 等空闲超时时间 idle_timeout 以秒为单位，超过设置的空闲超时时间后该流规则将被自动删除，空闲超时时间设置为 0 表示该流规则永不过期，idle_timeout 将不包含于 ovs-ofctl dump-flows brname 的输出中。


条件字段包括:

- 输入端口号 in_port

- 源目的 mac 地址 dl_src/dl_dst

- 源目的 ip 地址 nw_src/nw_dst

- 数据包类型 dl_type

- 网络层协议类型 nw_proto 
  
这些字段可以任意组合，但在网络分层结构中底层的字段未给出确定值时上层的字段不允许给确定值，即一
条流规则中允许底层协议字段指定为确定值，高层协议字段指定为通配符(不指定即为匹配任何值)，而不允许高层协议字段指定为确定值，
而底层协议字段却为通配符(不指定即为匹配任何值)，否则，ovs-vswitchd 中的流规则将全部丢失，网络无法连接。

动作字段包括正常转发 normal、定向到某交换机端口 output：port、丢弃 drop、更改源目
的 mac 地址 mod_dl_src/mod_dl_dst 等，一条流规则可有多个动作，动作执行按指定的先后顺序依次完成。


示例::

    #查看某网桥信息
    ovs-ofctl show br-tun

    #查看某网桥上所有端口的状态
    ovs-ofctl dump-ports br-tun

    #添加一条流表规则 丢弃从port2上发来的所有数据表
    ovs-ofctl add-flow br-tun idle_timeout=120,in_port=2,actions=drop

    #查看某网桥上面的流表规则
    ovs-ofctl dump-flows br-tun

    #屏蔽所有进入 OVS 的以太网广播数据包
    ovs-ofctl add-flow ovs-switch "table=0, dl_src=01:00:00:00:00:00/01:00:00:00:00:00, actions=drop"

    #屏蔽 STP 协议的广播数据包
    ovs-ofctl add-flow ovs-switch "table=0, dl_dst=01:80:c2:00:00:00/ff:ff:ff:ff:ff:f0, actions=drop"


Qos设置
=================
Qos可以针对网络接口，也可以针对端口设置::

    #针对网络接口  1000±100kbps
    ovs-vsctl set interface tap-xxx ingress_policing_rate=1000
    ovs-vsctl set interface tap-xxx ingress_policing_burst=100

官方参考
http://openvswitch.org/support/config-cookbooks/qos-rate-limiting/  

端口映射
=================
将发往 p0 端口和从 p1 端口发出的数据包全部定向到 p2 端口，用 ovs-vsctl list port 命令查看 p0、p1、p2 端口的 uuid 分别为id1、id2、id3::
    
    ovs-vsctl --set bridge br0 mirrors=@m-- --id=@m create mirror name=mymirror  \
    select-dst-port=id_1 \
    select-src-port=id_2 \
    output-port=id_3


其他设置
=================
屏蔽对目的主机访问::

    ovs-ofctl add-flow br0 idle_timeout=0,dl_type=0x0800,nw_src=xx.xx.xx.xx,actions=drop



参考资料
=================
http://www.ibm.com/developerworks/cn/cloud/library/1401_zhaoyi_openswitch/

http://openvswitch.org/support/

http://www.opencloudblog.com/?p=300


