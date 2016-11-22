=======================================
虚机镜像制作
=======================================
对于openstack无论你是在公有云还是私有云，创建虚机都需要系统镜像来做支撑，也就是glance组件所存放的镜像，镜像的制作是非常繁琐的，尤其是你要制作云环境下的镜像，我们称其为cloud-images，和普通的baremetal镜像不同，你需要在cloud-images内置一些组件如cloudinit等，windows还好说，但是Linux的发行版众多，每个都要制作非常麻烦。

openstack官方提供了一个手动制作镜像的文档 `creating_images_manually <http://docs.openstack.org/image-guide/content/ch_creating_images_manually.html>`_ ，如果每个发行版镜像都这么做，那是相当麻烦，而且你肯定有定制镜像的需求，即在镜像内置一些组件以便支持云平台的特性，比如可能会有需要镜像支持root密码修改功能或者需要内置qga等。每次镜像修改需求都要手工来一次，想想就恶心了。

官方也提供了自动构建镜像的方法 `creating_images_automatically <http://docs.openstack.org/image-guide/content/ch_creating_images_automatically.html>`_ ，但是里面提到的工具和方案，要么是已经长时间不维护的，要么是只针对某个发行版的，基本上没有提出一个工具构建全部镜像的方案，另外也无法做到对镜像的每次修改可以记录和回滚。

以上铺垫，引出本文讨论的内容，如何高度灵活的构建镜像。


我主要是采用了disk-image-builder来实现镜像制作，disk-image-builder是tripleO的子项目，用于给tripleO构建镜像之用，disk-image-builder采用element拼装这种灵活的架构，可以满足镜像制作时多变的需求，而且可以利用好各个linux发行版已有的基础镜像。

安装
=================
首先需要安装一些基本包，还有disk-image-builder依赖的项目::
    
    #安装dib-utils
    $ git clone https://github.com/openstack/dib-utils.git
    $ cd dib-utils && python setup.py install

    #安装diskimage-builder
    $ git clone https://github.com/openstack/diskimage-builder.git
    
    #添加环境变量
    export DIB_IMAGE_CACHE=<your image cache path>
    export PATH=<xxxxx/diskimage-builder/bin>:$PATH
    
    #其他依赖项
    $ apt-get install -y qemu-utils 
    $ apt-get install -y kpartx

这样基本安装就完成了，我们主要使用的命令是disk-image-create，可以加-h查看具体参数用法。

使用样例
=================
我们以创建ubuntu镜像为例，可以参考下面操作一样，非常简单::

    $ mkdir ubuntu && cd ubuntu
    
    #创建vm_profile文件 内容如下
    export DIB_RELEASE=trusty   # 可替换为其他ubuntu系统代号名称
    export DIB_DISTRIBUTION_MIRROR=http://cn.archive.ubuntu.com/ubuntu  #可替换其他mirror
    export DIB_CLOUD_INIT_DATASOURCES="ConfigDrive, OpenStack, NoCloud"    #cloud-init datasource配置
    $ source vm_profile

    #构建
    $ disk-image-create --offline -a amd64 -o ubuntu-trusty-amd64 vm ubuntu
    
注意，第一次构建时请把 --offline 去掉，第一次需要下载原始镜像，执行完毕后，会生成 一个ubuntu-trusty-amd64.qcow2。如果要raw镜像，请执行qemu-img进行转换。


自定义element
=================
disk-image-builder是灵活的element架构，你可以给你的镜像添加不同的element，制作镜像时你可以灵活的控制element是否加入到镜像中。我们以让镜像支持通过openstack注入root密码为例，原理请看之前的文档( :doc:`./inject_passwd`)。操作如下::

    $ cd elements && mkdir cloud-init-cfg
    $ cd elements/cloud-init-cfg && mkdir  install.d
    $ cd elements/cloud-init-cfg/install.d 
    
创建文件 05-set-cloud-cfg 并 chmod a+x，内容如下::

    #!/bin/bash
    set -eu
    set -o pipefail
    sed -i "s/disable_root: true/disable_root: false/" /etc/cloud/cloud.cfg
    sed -i "s/disable_root: 1/disable_root: 0/" /etc/cloud/cloud.cfg
    sed -i "/package_mirrors/,+13d" /etc/cloud/cloud.cfg
    cat > /etc/cloud/cloud.cfg.d/92-dib-cloud-neunn.cfg <<EOF
    chpasswd: { expire: False }
    EOF
     
    home=$(dirname $0)
    cloudinit_dir=`python -c "import os,cloudinit;print os.path.dirname(cloudinit.__file__)"`
    install -m 0644 -o root -g root $home/cc_set_passwd.patch $cloudinit_dir/cc_set_passwd.patch
    cd $cloudinit_dir && sudo patch -p2 < cc_set_passwd.patch
    rm -f $cloudinit_dir/cc_set_passwd.patch


创建patch文件，文件内容在之前提到的原理文档中。

如果你要在ubuntu镜像中支持root密码注入，那么执行下面的命令创建ubuntu镜像::

        
    $ disk-image-create --offline -a amd64 -o ubuntu-trusty-amd64 vm cloud-init-cfg ubuntu

其他镜像要使用，同理方法


