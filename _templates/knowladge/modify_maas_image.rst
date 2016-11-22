=======================================
修改maas的boot-image
=======================================
maas是canonical出品的裸机部署工具，这里就不做太多介绍，裸机部署肯定需要提供镜像，但是如果对maas自带镜像有改动需求，此文对你有用(本文适用maas1.5)

首先我们手动添加一个镜像::

    # 这是maas的镜像存放目录
    cd /var/lib/maas/boot-resources/current/amd64/generic
    
    # 一般自带一个trusty镜像
    # 我们要新增一个基于trusty修改的镜像
    cp -r ./trusty neunn
    cd neunn/release

    # root-tgz 是我们需要修改的

    # root-tgz 是从系统根目录打包的文件
    file root-tgz
    # root-tgz: gzip compressed data, from Unix

修改root-tgz，比如我们修改镜像，让其root账户有个默认密码::

    # 拷贝root-tgz到操作目录
    cp root-tgz /opt/images/

    # 解压
    tar xzf root-tgz

    # 修改root密码 编辑 ./etc/shadow 文件

    # 重新压缩
    tar czvf root-tgz-new *

    # 替换原来的镜像文件

让镜像在maas-UI上能够显示出来::

    vim /usr/lib/python2.7/dist-packages/maasserver/enum.py
    # 修改如下
    
    class DISTRO_SERIES:
    """List of supported ubuntu releases."""
    default = ''
    trusty = 'trusty'
    # add by niusmallnan
    neunn = 'neunn'
    
    DISTRO_SERIES_CHOICES = (
        (DISTRO_SERIES.default, 'Default Ubuntu Release'),
        #(DISTRO_SERIES.precise, 'Ubuntu 12.04 LTS "Precise Pangolin"'),
        #(DISTRO_SERIES.quantal, 'Ubuntu 12.10 "Quantal Quetzal"'),
        #(DISTRO_SERIES.raring, 'Ubuntu 13.04 "Raring Ringtail"'),
        #(DISTRO_SERIES.saucy, 'Ubuntu 13.10 "Saucy Salamander"'),
        (DISTRO_SERIES.trusty, 'Ubuntu 14.04 LTS "Trusty Tahr"'),
        (DISTRO_SERIES.neunn, 'Ubuntu Neunn Ncloud-Trusty'),
    )

重启maas服务::

    service tgt                 restart
    service apache              reload
    service maas-cluster-celery restart
    service maas-dhcp-server    restart
    service maas-pserv          restart
    service maas-region-celery  restart
    service maas-txlongpoll     restart

最后一步不要忘记，要在UI上点击一下 "Import boot Images"，等待稍许片刻，
新增的镜像就会导入到maas中。














