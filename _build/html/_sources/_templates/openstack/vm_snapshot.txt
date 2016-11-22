=======================================
虚拟机的快照
=======================================





snapshot技术分类
=================
Internal snapshot: A type of snapshot, where a single QCOW2 file will hold both the ‘saved state’ and the ‘delta’ since that saved point. ‘Internal snapshots’ are very handy because it’s only a single file where all the snapshot info. is captured, and easy to copy/move around the machines.  （管理方便）

External snapshot: Here, the ‘original qcow2 file’ will be in a ‘read-only’ saved state, and the new qcow2 file(which will be generated once snapshot is created) will be the *delta* for the changes. So, all the changes will now be written to this delta file. ‘External Snapshots’ are useful for performing backups. Also, external snapshot creates a qcow2 file with the original file as its backing image, and the backing file can be /read/ in parallel with the running qemu.（子快照依赖于父快照，占用磁盘空间小，适用于备份场景）::
    
    .-----------.   .-----------.   .------------.  .------------.  .------------.
    |           |   |           |   |            |  |            |  |            |
    | RootBase  |<--- Overlay-1 |<--- Overlay-1A <--- Overlay-1B <--- Overlay-1C |
    |           |   |           |   |            |  |            |  | (Active)   |
    '-----------'   '-----------'   '------------'  '------------'  '------------'
            ^
            |
            |       .-----------.    .------------.
            |       |           |    |            |
            '-------| Overlay-2 |<---| Overlay-2A |
                    |           |    | (Active)   |
                    '-----------'    '------------'



在openstack中只提供Internal snapshot方式，并支持live snapshot和cold snapshot。

nova中的实现
=================

在 nova/virt/libvirt/driver.py ::

     if state == power_state.SHUTDOWN:
            live_snapshot = False

        # NOTE(dkang): managedSave does not work for LXC
        if CONF.libvirt.virt_type != 'lxc' and not live_snapshot:
            if state == power_state.RUNNING or state == power_state.PAUSED:
                self._detach_pci_devices(virt_dom,
                    pci_manager.get_instance_pci_devs(instance))
                virt_dom.managedSave(0)

        snapshot_backend = self.image_backend.snapshot(disk_path,
                image_type=source_format)

        if live_snapshot:
            LOG.info(_("Beginning live snapshot process"),
                     instance=instance)
        else:
            LOG.info(_("Beginning cold snapshot process"),
                     instance=instance)

        update_task_state(task_state=task_states.IMAGE_PENDING_UPLOAD)
        snapshot_directory = CONF.libvirt.snapshots_directory
        fileutils.ensure_tree(snapshot_directory)
        with utils.tempdir(dir=snapshot_directory) as tmpdir:
            try:
                out_path = os.path.join(tmpdir, snapshot_name)
                if live_snapshot:
                    # NOTE(xqueralt): libvirt needs o+x in the temp directory
                    os.chmod(tmpdir, 0o701)
                    self._live_snapshot(virt_dom, disk_path, out_path,
                                        image_format)
                else:
                    snapshot_backend.snapshot_extract(out_path, image_format)
 

虚拟机如果是运行状态，快照是很慢的，而且宿主机需要有足够的空间(至少要比vm的root disk大才行)。

虚拟机如果是关闭状态，快照是非常快的，他只是vm的disk文件做了格式转换，转成qcow2。



参考资料
=================
1. http://blog.csdn.net/halcyonbaby/article/details/19998155
2. http://blog.csdn.net/duan101101/article/details/19424479



















