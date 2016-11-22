=======================================
健康检查与自动化测试
=======================================
当你搭建完一整套openstack环境，你希望对他进行一个健康检查，这个检查不仅仅局限于进程服务级别，你希望深入到组件内部调用机制上，比如真正去创建vm并测试他网络的连通性。

当你在开发时，对openstack打了一些自己开发的补丁，需要测试一下整体功能是否有影响，你需要跑一些自动化测试的用例，而且他能基本覆盖openstack的功能。

正常人面对这种需求有两种选择，自己开发或者使用开源项目，自己开发的难道并不高，无非是编写测试用例，问题在于你要理顺openstack每个功能，结合业务场景编写测试用例还是破费心思的。

再看开源项目，经过一些调研，发现Mirantis开源的ostf框架颇为不错，而且在他们的部署工具Fuel中应用，非常实用的一个功能。


运行ostf
=================
ostf是openstack孵化项目，在stackforge中可以找到 `代码库 <https://github.com/stackforge/fuel-ostf>`_ ，值得注意的是ostf的版本是跟Fuel一起走的。

先克隆一份代码到本地，切换到需要的稳定分支上，构建python虚拟环境，安装依赖包，这些都不会先去自学基础知识，过程就不赘述了。

工程主要包括两大目录，fuel_health 和 fuel_plugin，前者是健康检查功能真正实现者，后者主要是通过API方式把ostf集成到Fuel中。我们抛开fuel_plugin，如果生产环境不用Fuel部署，这个东西对我们来说意义不大。

运行fuel_health很简单，只需切到fuel_health目录执行::

    export CUSTOM_FUEL_CONFIG='#这里替换你的test.conf的路径#'
    python -m unittest2 -v

fuel_health的本质上就是python unitest，所以运行起来并不麻烦，最大的坑在于test.conf的配置非常不完整，你需要通过不断的调试，来把test.conf的配置补充完整::
    
    #比如
    [compute]
    compute_nodes = 192.168.250.49, 192.168.250.28, 192.168.250.21
    libvirt_type = kvm

除此之外，fuel_health默认会对heat、sahara、murano等不常用组件做健康检查，如果你的环境不需要这些组件，那么你需要在fuel_health代码中去掉这些功能，当然因为这是unitest，所以你也可以有选择的执行测试用例，比如::
    
    #执行ceilometer测试用例
    export CUSTOM_FUEL_CONFIG='#这里替换你的test.conf的路径#'
    python -m unittest2 tests.platform_tests.test_ceilometer



ostf会做什么
=================
ostf会真正做业务逻辑来测试功能，以test_volume_create为例，他会做如下检测::
    
     def test_volume_create(self):
        """Create volume and attach it to instance
        Target component: Compute

        Scenario:
            1. Create a new small-size volume.
            2. Wait for volume status to become "available".
            3. Check volume has correct name.
            4. Create new instance.
            5. Wait for "Active" status
            6. Attach volume to an instance.
            7. Check volume status is "in use".
            8. Get information on the created volume by its id.
            9. Detach volume from the instance.
            10. Check volume has "available" status.
            11. Delete volume.
            12. Verify that volume deleted
            13. Delete server.
        Duration: 350 s.
        """

这样的深入检测才是有意义的。
