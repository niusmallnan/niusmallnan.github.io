.. niusmallnan documentation master file, created by
   sphinx-quickstart on Tue Feb 18 13:49:43 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


=======================================
巧用flavor metadata决定虚机的宿主机
=======================================
关于创建虚拟机，收到一个需求，是希望通过单纯地指定flavor来确定该虚机被创建的区域，于是做了些小研究，阅读了些源码，记录一下。

实现原理
======================
如果不知道nova如何确定在哪台Host上创建虚机的过程，请查看这篇文档 :doc:`host_aggregates` ，
读完后，您的脑海中应该会有一个filter_properties的概念，这个对物理Host的选择，
起到至关重要的作用。那么它的属性都是哪些因素决定的呢？

阅读源码得知，filter_properties的组成大致如下::

    instance_type : 调用get_flavor_by_flavor_id 和传入的flavor相关

    availability_zone : nova boot 时传入的 --availability-zone
    
    scheduler_hint: nova boot 时传入的 --hint

get_flavor_by_flavor_id 返回的就是flavor的extra_specs属性，如此我们便可通过flavor来改变filter_properties，进而控制物理Host的选择策略。

filter_properties可以改变了，但是我们还差左后一步，我们需要一个Filter来处理传入的参数。
好在，强大的openstack社区已经给我们准备好了，只不过这个Filter并不是默认生效的，您需要在配置中开启，了解他的原理请看scheduler/filters/aggregate_instance_extra_specs.py的代码。



实践方法
======================
创建一个新的flavor命名为test，给extra_specs添加一个availability_zone属性，
一系列操作后这个flavor的detail大致是这样的::

    $ nova flavor-show 88
    +----------------------------+----------------------------------+
    | Property                   | Value                            |
    +----------------------------+----------------------------------+
    | name                       | test                             |
    | ram                        | 1024                             |
    | OS-FLV-DISABLED:disabled   | False                            |
    | vcpus                      | 2                                |
    | extra_specs                | {u'availability_zone': u'horde'} |
    | swap                       | 128                              |
    | os-flavor-access:is_public | True                             |
    | rxtx_factor                | 1.0                              |
    | OS-FLV-EXT-DATA:ephemeral  | 0                                |
    | disk                       | 5                                |
    | id                         | 88                               |
    +----------------------------+----------------------------------+


建立一个availability_zone名为horde，并关联主机compute2，最终的效果是::
    
    $ nova-manage service list
    Binary           Host              Zone
    nova-compute     compute1          nova
    nova-compute     compute2          horde

修改nova.conf的scheduler_default_filters配置，``增加AggregateInstanceExtraSpecsFilter，但同时要注意去掉ComputeCapabilitiesFilter``，
否则哥俩会有冲突导致不能过滤出可用的Host，可以参考下面的配置::
        
    修改前是默认配置
    scheduler_default_filters=RetryFilter,AvailabilityZoneFilter,RamFilter,ComputeFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter

    修改后
    scheduler_default_filters=RetryFilter,AvailabilityZoneFilter,RamFilter,ComputeFilter,ImagePropertiesFilter,AggregateInstanceExtraSpecsFilter

修改完后需要重启nova-scheduler服务，然后就可以选用这个flavor创建虚机，
最终你会看到这个flavor创建的虚机都会建到指定的zone上。

Nova组件有很多不错的功能，官方文档并没有写得很详细，需要好好研读代码，
才能挖掘出来。









    








