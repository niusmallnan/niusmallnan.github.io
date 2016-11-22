=======================================
Salt问题及技巧整理
=======================================
整理一些工作中遇到的问题以及好的技巧

如何指定机器
=================
看这里::

    salt '*' test.ping -v
    salt 'Cloud[1-5]' test.ping -v
    salt 'test*' test.ping -v
    salt -E 'Test.*' test.ping -v
    salt -L 'TestVM01,TestVM02' test.ping -v #中间不要有空格
    salt -G 'os:CentOS' test.ping -v #同样的，不要有多余字符
    salt -N 'group1' test.ping -v
    salt -S '127.0.0.1' test.ping -v #minion只要在这个子网就会匹配
    salt -C 'E@TestVM0.* and G@os:CentOS' test.ping -v #允许逻辑运算


