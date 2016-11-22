=======================================
基于ceph快速创建虚机
=======================================
基于ceph来构建openstack的统一存储是目前非常流行的做法，更加详细的说明，可以参考 `openstack与ceph整合 <https://www.ustack.com/blog/openstack_and_ceph/>`_ 。

openstack提供了5种boot虚机的方式，当然这也是时下各种云平台通用的做法，分别为:

- 从镜像启动
- 从快照启动
- 从云硬盘启动
- 从镜像启动（创建一个新卷）
- 从云硬盘快照启动（创建一个新卷）
   
我们所知rbd是支持copy-on-write的，这个特性可以体现在openstack的代码中调用了rbd clone这个命令，当你像复制一个非常大的卷时，rbd clone是秒级的，与之相对的做法是，重新创建一个raw文件然后上传给ceph，这明显是非常耗时的。

如果我们的nova和glance都是配置使用了ceph的rbd，那么理想状况下，我们创建一台虚机，虚机的根磁盘应该是通过glance中对应image进行rbd clone而得。

在Juno版本中，这的确已经实现，但是I版并非如此，本文主要介绍下讨论的便是在I版如何支持该功能。

社区的贡献
=======================
openstack社区是非常强大的，早在13年就有人提出此bug，经过一系列讨论和具体实现，已经完全实现了此功能 `rbd-clone-image-handler <https://review.openstack.org/#/c/94295/>`_ 。当然这些代码在Juno版本才合入master，如果想在I版使用，你可以基于I版手动合并这些patch。

对nova代码不太深入理解的，可能觉得合并patch有些困难，建议你使用网友合并好的icehouse分支 `stable-icehouse <https://github.com/angdraug/nova/tree/rbd-ephemeral-clone-stable-icehouse>`_ 。

当然你需要一些额外的配置，更多的配置请参考 `ceph官网 <http://docs.ceph.com/docs/next/rbd/rbd-openstack/>`_ 。











