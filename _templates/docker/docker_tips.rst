=======================================
docker问题及技巧整理
=======================================
整理一些工作中遇到的问题以及好的技巧

如何在container中运行crontab
==============================
看这里
http://stackoverflow.com/questions/20545554/how-do-i-start-cron-on-docker-ubuntu-base


进入container
==============================
在docker1.3以前我们可以使用nscenter或者ssh，而之后的版本我们可以直接::
    
    docker exec -it CONTAINER_NAME /bin/bash 

container中实现多网卡
==============================
第一种方式就是我们使用host网络模式(默认是bridge模式)，这种可以映射全部物理主机网卡到容器中，那么在创建容器的时候需要这样::

    docker run --net=host --name=test -idt ubuntu:trusty /bin/bash

第二种就是使用pipework，一个集成了网络配置命令的的脚本，操作容器的net namespace::

    # https://github.com/jpetazzo/pipework
    E.g. 将主机上的物理网卡添加到容器内，这样容器将监听物理网卡
    pipework eth1 container_name 9.81.1.183/16
    pipework eth1 container_name DHCP
    pipework eth1 container_name 0/0

    E.g. 给容器添加一个新的网卡，这个网卡连接到自定义的网桥上
    pipework br0 container_name 9.81.1.166/16
    pipework ovsbr0 container_name 9.81.1.166/16

docker 国内快速安装
==============================
使用daocloud提供的源::

    $ curl -sSL https://get.daocloud.io/docker | sh
