.. niusmallnan documentation master file, created by
   sphinx-quickstart on Tue Feb 18 13:49:43 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


=======================================
SDN测试方案
=======================================
东网云平台采用SDN(软件定义网络)技术，以实现云平台的网络方案。本方案主要目
的是构建一个测试方案，以便我们有足够的依据参考来完善SDN的各方面性能。


拓扑结构
=================
我们的SDN方案，主要采用openstack的neutron组件来完成，然后做个LB看起来完善一点，整体的网络拓扑结构应该是这样的： 

.. image:: /images/sdn_test_tuopu.jpg

假设我们会建立这样几个租户，他们分配的网段和隔离关系是这样的：

.. image:: /images/sdn_test_tenant.png

我们使用vlan模式，对于openstack而言，几个重要的节点间的关系，是这样的：

.. image:: /images/sdn_test_nodes.png


功能测试
================
- 租户隔离
- 租户和管理节点的隔离


重要性能指标
=================
主要测试的网络指标是下面两个:

- 带宽，单位时间内从网络中的某一点到另一点所能通过的"最高数据率",单位为b/s
- 时延，是数据(一个报文或分组)从网络的一段传送到另一端所需要的时间



测试方法
=================
我们考虑使用的测试工具是Iperf，它是一个网络性能测试工具。可以测试TCP和UDP
带宽质量，可以测量最大TCP带宽，具有多种参数和UDP特性，可以报告带宽，延迟抖
动和数据包丢失。

网友总结的使用方法如下:

http://ponyjia.blog.51cto.com/917324/830800











