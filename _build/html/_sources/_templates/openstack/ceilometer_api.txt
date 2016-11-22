=======================================
ceilometer的API使用
=======================================


基本用法
=================================
下面用命令行方式表达，如果要查看详细请求，请加--debug参数即可。假设某台vm的uuid为f3bc23e4-5821-462d-bdf2-476275f76a79，vm运行时间的监控项meter是instance。

先来看关于sample采样点的相关API::

    #获取该vm运行时间的全部采样点
    ceilometer sample-list -m instance -q resource_id=f3bc23e4-5821-462d-bdf2-476275f76a79

    #获取该vm运行时间的最近10个采样点
    ceilometer sample-list -m instance -q resource_id=f3bc23e4-5821-462d-bdf2-476275f76a79 -l

    #获取该vm运行时间的两个时间戳内的采样点
    ceilometer sample-list -m instance -q \
        "resource_id=f3bc23e4-5821-462d-bdf2-476275f76a79;timestamp>2014-09-21T03:18:20;timestamp<2014-09-21T03:30:20"
    
再来看看statistics统计的相关API::
    
    #vm运行总时间 Duration字段就是运行的总时间
    ceilometer statistics -m instance -q "resource_id=f3bc23e4-5821-462d-bdf2-476275f76a79"
    +--------+----------------------------+----------------------------+-------+-----+-----+------+-----+----------+----------------------------+----------------------------+
    | Period | Period Start               | Period End                 | Count | Min | Max | Sum  | Avg | Duration | Duration Start             | Duration End               |
    +--------+----------------------------+----------------------------+-------+-----+-----+------+-----+----------+----------------------------+----------------------------+
    | 0      | 2014-09-22T02:51:44.337000 | 2014-09-22T02:51:44.337000 | 39    | 1.0 | 1.0 | 39.0 | 1.0 | 19453.46 | 2014-09-22T02:51:44.337000 | 2014-09-22T08:15:57.797000 |
    +--------+----------------------------+----------------------------+-------+-----+-----+------+-----+----------+----------------------------+----------------------------+

    #某个时间段内vm的运行时间
    ceilometer statistics -m instance -q "resource_id=f3bc23e4-5821-462d-bdf2-476275f76a79" \
         -q "resource_id=f3bc23e4-5821-462d-bdf2-476275f76a79;timestamp>2014-09-22T07:18:20;timestamp<2014-09-22T08:18:20"


参考:

#. `ceilometer命令行使用 <http://yansu.org/2013/05/01/terminal-command-of-ceilometer.html>`_
#. `ceilometer Statistics <https://support.rc.nectar.org.au/docs/ceilometer-statistics>`_






