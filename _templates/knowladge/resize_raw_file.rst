=======================================
使用qemu-img改变镜像文件大小
=======================================
有时我们的raw文件比如虚机镜像比较大，但是它的disk size却不是很大，我们需要给他做一些精简。

首先查看下raw文件的信息::

    $ qemu-img info test.raw
    image: test.raw
    file format: raw
    virtual size: 20G (21474836480 bytes)
    disk size: 1.1G

可以看出他的真实大小只有1.1G，如果把其转换成qcow2格式，那边一目了然::

    $ qemu-img convert -f raw test.raw -O qcow2 test.qcow2
    $ qemu-img info test.qcow2
    image: test.raw
    file format: qcow2
    virtual size: 20G (21474836480 bytes)
    disk size: 1.1G
    cluster_size: 65536
    Format specific information:
        compat: 1.1
            lazy refcounts: false
    $ du -h test.qcow2
    1.1G    test.qcow2

virtual_size就是qcow2转换成raw后，这个文件的大小，raw文件大小改变需要这样::

    $ qemu-img resize test.raw -5G
    Image resized.
    $ qemu-img info test.raw
    image: test.raw
    file format: raw
    virtual size: 15G (16106127360 bytes)
    disk size: 1.1G
    $ du -h test.raw
    15G    test.raw

    $ qemu-img resize test.raw +5G
    Image resized.
    $ qemu-img info test.raw
    image: test.raw
    file format: raw
    virtual size: 20G (21474836480 bytes)
    disk size: 1.1G
    $ du -h test.raw
    20G    test.raw

如果是qcow2格式的文件，可以使用shrink来减容::

    $ qemu-img convert -c -O qcow2 source.qcow2 shrunk.qcow2


