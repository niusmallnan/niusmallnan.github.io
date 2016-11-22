=======================================
提升虚机创建后的启动速度
=======================================
虚机创建后并首次启动时，会进行元数据注入操作，可以有两种方式cloud-init与guestfs，guestfs方式非常不灵活受限很大，
我们之前在这里（ :doc:`./inject_passwd` ）探讨过一些，而cloud-init可以有多种方式进行元数据注入，
之前有讨论过EC2的方式（ :doc:`./metadata_server` ），本文主要侧重cloud-init另外一种常用方式config-drive。

对于元数据注入的操作，如果使用cloud-init的EC2方式是非常耗时的，因为它要走网络请求，而且一个请求是被多次转发才能到nova-api处理，
而使用config-drive方式则是相对省时，大大提升创建后的启动速度。

原理以及实现
=================
使用config drive有两种方式实现，可以通过 boot vm 时加入参数::

    # set config-drive true
    $ nova boot --config-drive true --image my-image-name --key-name mykey --flavor 1 \
        --user-data ./my-user-data.txt myinstance --file /etc/network/interfaces=/home/myuser/instance-interfaces \
        --file known_hosts=/home/myuser/.ssh/known_hosts --meta role=webservers --meta essential=false

也可以在计算节点上做配置，强制使用config-drive，修改/etc/nova/nova.conf ::
    
    force_config_drive=true

更加详细的说明，可以参考 `官方文档 <http://docs.openstack.org/user-guide/content/enable_config_drive.html>`_

设置完毕，创建虚机，libvirt.xml 文件会多出一段描述::

    <disk type="file" device="cdrom">
        <driver name="qemu" type="raw" cache="none"/>
        <source file="/var/lib/nova/instances/37bf37e7-fd38-4b31-aa1b-5dffa0c35abe/disk.config"/>
        <target bus="ide" dev="hdd"/>
    </disk>

libvirt会在instance对应的目录下创建disk.config，里面的内容就是虚机对应的元数据，这个会作为一个cdrom设备挂载到虚机中，
然后cloud-init会加载它并读取对应的内容，将元数据注入到虚机中，当然cloud-init也需要简单配置下::

    #添加如下配置，优先使用config-drive
    datasource_list: ['ConfigDrive', 'Ec2', 'NoCloud']
    datasource:
       Ec2:
            timeout: 2
            max_wait: 5


一切设定完毕后，创建新的镜像然后启动虚机，我们以ubuntu-server-1204为例，启动时间大概是10s左右。

其它优化项
=================
对于ubuntu-server-1404，有一个启动非常耗时的组件pollinate，pollinate是1404新出的安全组件，pollinate是在云环境中用于提供随机数生成器的种子，
但是他在启动时会到 entropy.ubuntu.com 这个域名去申请seed，而且还是走https请求，速度相当慢，去掉该组件后启动速度会有质的飞跃。

看看这里老外兄弟是如何吐槽pollinate `pollinate is stupid <http://pastebin.com/t1wFnJZY>`_

但是去掉后会不会影响，可以看看nova的源码关于config-drive的实现，nova中会自动生成一个random_seed用于实现pollinate同样的功能::

    # metadata/base.py
    def _metadata_as_json(self, version, path):
        metadata = {'uuid': self.uuid}
        if self.launch_metadata:
            metadata['meta'] = self.launch_metadata
        if self.files:
            metadata['files'] = self.files
        if self.extra_md:
            metadata.update(self.extra_md)
        if self.network_config:
            metadata['network_config'] = self.network_config
        if self.instance['key_name']:
            metadata['public_keys'] = {
                self.instance['key_name']: self.instance['key_data']
            }
            metadata['hostname'] = self._get_hostname()
            metadata['name'] = self.instance['display_name']
            metadata['launch_index'] = self.instance['launch_index']
            metadata['availability_zone'] = self.availability_zone
            
            #看这里 看这里
            if self._check_os_version(GRIZZLY, version):
                metadata['random_seed'] = base64.b64encode(os.urandom(512))

        return json.dumps(metadata)

