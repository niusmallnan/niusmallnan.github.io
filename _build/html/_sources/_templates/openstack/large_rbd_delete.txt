=======================================
cinder中大块rbd删除问题
=======================================
我们知道cinder支持很对driver，rbd是集成度最好功能也相对齐全的driver，但是在使用中，我们会发现一个问题，就是large size的rbd会删除非常慢，而且这个删除过程会block掉cinder-volume的服务，导致新的cinder请求通过schedule时不能转到这个cinder-volume上。

这个问题ceph官方也认可，主要原因是librbd对rbd的操作是同步操作会阻塞当前进程，而且ceph官方也没有修改计划，毕竟改成异步操作是大改动啊。

所以我们需要在cinder层面上修复这个问题，既然会阻塞当前进程，那么我们做删除操作时，应该启一个子进程来执行这个操作，阻塞也只会阻塞子进程，不会影响cinder-volume的服务。


修复方式
===================
社区也有人提出了这个问题，有人提出了修改代码，但是目前还没有被合并，code review一直没有通过，但是功能是好用的，review没过的原因主要是代码太糙。

这个问题的review地址是 `review-145678 <https://review.openstack.org/#/c/145678/>`_ ，主要修改 cinder/volume/drivers/rbd.py 文件内容，如下::

     if clone_snap is None:
         LOG.debug(_("deleting rbd volume %s") % (volume_name))

         # RBD.remove from rbd python binding will be blocked in
         # the librbd::remove() and cannot be monkey patched by
         # evenlet. This will block all the rest greenthread from
         # running in the volume process until returning back from
         # RBD.remove().
         # librados/librbd doesn't work in the forked process.
         # so running RBD.remove from a child process won't be an
         # option.
         # Before ceph community provides an async RBD.remove(), the
         # only viable option is to execute rbd cli from child process

         args = ['rbd', '--pool',
                 self.configuration.rbd_pool,
                 'rm', volume_name]
         args.extend(self._ceph_args())
         try:
             self._try_execute(*args)
         except Exception:   
             .....
             ....
             ...
             ..
             .

_try_execute 便会启动子进程去执行rbd rm的操作。












