=======================================
docker部署OpenStack API
=======================================
docker的优势好处自然不必多说，本文主要讨论基于docker这个轻量级的容器部署OpenStack API 的方案。我们知道OpenStack中controller node 需要安装keystone、nova、cinder等api组件，当然我们部署生产环境时，往往会把这些组件分离到单独的物理机上。我们自动化安装时，会在这些物理机上执行安装各种包的脚本，它不够灵活，不能满足未来OpenStack升级的需求，而且安装过程一旦出错回滚起来非常麻烦，可能涉及到要删除之前的包和之前的配置。

为了满足灵活多变的需求，和安装过程的清爽简洁，我们引入了docker来部署OpenStack API。PS: 试用过后，感叹docker如此惊艳！


基础知识
=================
docker的基础知识本文不在讨论之列，为了引导后面的内容，还需简单介绍两个重要概念，镜像 & 容器。

docker的镜像(images)就像kvm hyper-v等虚拟化镜像类似，但是与其他真正虚拟化技术不同的是，docker的镜像是非常灵活的，你可以自己制作镜像，也可以去docker hub 上pull下来，因为docker是应用级的容器，所以docker hub 上很多类似mysql、mencached、mongodb等应用镜像，比如我们本机的镜像有如下::

    $ docker images 
    root@docker-1:~# docker images 
    REPOSITORY                 TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    niusmallnan/ops-base       0.1                 384eb099ca94        23 hours ago        474 MB
    tutum/haproxy              latest              4a3f66e6a4e6        5 weeks ago         455.8 MB
    tutum/rabbitmq             latest              7ba95b1b0b7c        6 weeks ago         299.7 MB
    tutum/mysql                latest              3a32fd5b15e4        8 weeks ago         407.7 MB
    tutum/tomcat               8.0                 26fbe4eae01f        11 weeks ago        861.7 MB
    tutum/ubuntu               trusty              39020797ff4f        11 weeks ago        383 MB
    tutum/mongodb              latest              681730a9e4cf        12 weeks ago        637.8 MB
    tutum/memcached            latest              e6e1a7845b56        12 weeks ago        422.8 MB
    crosbymichael/dockerui     latest              109c8f1f632d        4 months ago        398.7 MB


他的镜像就如同git一样具有版本管理功能，你可以修改运行中的容器，然后commit方式保存镜像。使用脚本方式制作镜像也是非常灵活的，以ops-base镜像为例::

    FROM tutum/ubuntu:trusty
    MAINTAINER niusmallnan <zhangzb@neunn.com>

    RUN echo 'deb http://cn.archive.ubuntu.com//ubuntu trusty main restricted universe multiverse' > /etc/apt/sources.list
    RUN echo 'deb http://cn.archive.ubuntu.com//ubuntu trusty-updates main restricted universe multiverse' >> /etc/apt/sources.list
    RUN echo 'deb http://cn.archive.ubuntu.com//ubuntu trusty-security main restricted universe multiverse' >> /etc/apt/sources.list
    RUN echo 'Acquire::HTTP::Proxy "http://192.168.250.1:8000/";' > /etc/apt/apt.conf.d/90curtin-aptproxy

    RUN apt-get -y update

    RUN DEBIAN_FRONTEND=noninteractive apt-get -y install curl vim
    RUN curl https://pypi.python.org/packages/source/s/setuptools/setuptools-1.1.6.tar.gz | (cd /root;tar xvzf -;cd setuptools-1.1.6;python setup.py install)
    RUN easy_install pip
    RUN rm -rf /root/setuptools-1.1.6

我们以tutum/ubuntu:trusty 为基础镜像，然后修改了apt-get soucelist，并安装了vim curl pip等组件，这样新的镜像就包含了这些组件。

容器(container)是将镜像运行起来，具有网络、文件系统等，通过docker ps可以查看到::
    
    $ docker ps 
    CONTAINER ID        IMAGE                           COMMAND                CREATED             STATUS              PORTS                    NAMES
    ea749f708982        tutum/mysql:latest              /run.sh                24 hours ago        Up 24 hours         0.0.0.0:3306->3306/tcp   lonely_jones        
    8704dbf876bd        crosbymichael/dockerui:latest   ./dockerui -e /docke   5 days ago          Up 5 days           0.0.0.0:9000->9000/tcp   high_almeida   

tutum/mysql 是一个mysql应用容器，crosbymichael/dockerui 是基于docker api开发的一个web界面的 docker管理工具。运行一个容器，以mysql为例::

    $ docker run -d -p 3306:3306 -e MYSQL_PASS="admin" tutum/mysql

这是将本机的3306端口映射到container中的3306端口，并设置mysql用户密码为admin ::

    $ iptables -n -L
    .......
    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination         
    ACCEPT     tcp  --  0.0.0.0/0            172.17.0.99          tcp dpt:3306
    ACCEPT     tcp  --  0.0.0.0/0            172.17.0.64          tcp dpt:9000
    .......

    $ mysql -uadmin -padmin -h192.168.250.43 -P3306   # 192.168.250.43是 docker 容器所在的 物理机ip
    Your MySQL connection id is 1
    Server version: 5.5.37-0ubuntu0.14.04.1 (Ubuntu)
    mysql>


OpenStack API部署
=================
回到我们最初讨论的针对OPS API部署的需求，一是要简洁清爽，避免安装出错要回滚清包删配置的悲剧；二是要能满足未来升级OPS的便捷。如此，我们可以把安装过程打包
成docker的镜像，将这个镜像调试稳定，升级OPS时我们可以把新版本OPS的镜像制作出来并运行在容器里，切换新版本的API，一旦出错，我们可以很便捷的停掉新的容器，把老版本的容器启动起来，这样就可以做到可控的升级。


OPS API 的镜像制作，我们以keystone组件为例，新建一个目录，并在该目录下创建Dockerfile文件::

    #Dockerfile 的内容
    FROM ubuntu:trusty
    MAINTAINER niusmallnan <zhangzb@neunn.com>
    ENV DEBIAN_FRONTEND noninteractive
    RUN apt-get -y update && apt-get -y install keystone
    ADD service.sh /service.sh
    RUN chmod 755 /service.sh
    VOLUME ["/etc/keystone","/var/lib/keystone", "/var/log/keystone"]
    CMD ["/service.sh"]

    #service.sh 的内容
    #! /bin/bash
    set -e
    echo "Starting keystone."
    exec /usr/bin/keystone-all

    # build image
    $ docker build -t="keystone:icehouse" .

    $ docker images
    REPOSITORY                 TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    keystone                   icehouse            be352e1373eb        24 hours ago        292.1 MB

创建keystone服务容器，服务启动关闭，配置管理，日志管理，可以参考如下(建议docker1.6)::
    
    #config 这步是必须的 否则keystone将会因为没有配置文件导致无法启动 进而container也启动失败
    #/opt/keystone/conf 目录下添加keystone配置

    #Starting
    #日志目录 配置目录 服务专有目录 都映射本地
    $ docker run --net=host --name=keystone -idt -v /opt/keystone/log:/var/log/keystone -v /opt/keystone/conf:/etc/keystone -v /opt/keystone/lib:/var/lib/keystone keystone:icehouse

    #container启动 keystone服务一起启动
    #docker restart <container>  那么keystone服务就自动重启

    #修改配置都可以在宿主机上做

    #想要进入容器
    $ docker exec -it keystone /bin/bash


