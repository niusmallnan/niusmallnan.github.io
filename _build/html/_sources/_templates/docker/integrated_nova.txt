=======================================
docker与nova集成
=======================================
我们知道OpenStack的nova组件可以集成很多虚拟化技术，诸如kvm、hyper-v、vmware等，docker可以说是一种另类的“虚拟化”，OpenStack和docker作为时下最热门的两大开源项目，他们必须默默的牵手走在一起。


集成过程
=================
比较简单，没什么坑，安装官方的wiki做基本没有问题。

需要指出的是，nova-network 和 neutron 这两种网络模式，nova-docker都是支持的。

参考资料: 

https://wiki.openstack.org/wiki/Docker

问题整理
=================
在集成过程中，发现了一些坑，这里整理一下希望对大家有用:

- 如果使用vxlan网络发现container不能通过floating ip访问(ssh)，你需要把container的网卡mtu设置小一点，一般为1400，由于docker使用的必须是lxc的网络模式，所以网卡就是在宿主机上对应的namespace下面。

- 如果发现无法创建container且原因是无法将glance的镜像装载到计算节点本地，这是因为一个nova-docker driver的一个bug，镜像下载到本地后，需要load到本地的docker-images，load的时候nova-docker请求了错误的API(参数有误)，建议你直接手动把镜像load到本地即可。










