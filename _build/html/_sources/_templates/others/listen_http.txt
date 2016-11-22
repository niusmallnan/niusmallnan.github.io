=======================================
抓取Http数据包
=======================================
我们在做API开发时，经常会用到Http协议，在各方联调时我们总会需要在服务端输出以便于调试。

在不影响已有应用服务的前提下，我们最好有一个不用改代码不用加log的方式。

这里介绍两个比较好用的开源项目，当然数据收集都靠tcpdump::

    # https://github.com/xiaxiaocao/pcap-parser
    tcpdump -w- tcp port 8004 | parse_pcap -vv

    # https://github.com/lonkil/pyhttpcap






