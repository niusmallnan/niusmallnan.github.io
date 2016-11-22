=======================================
多DHCP Agent方案
=======================================

仔细阅读官方文档 http://docs.openstack.org/admin-guide-cloud/content/app_demo_multi_dhcp_agents.html

1、可以给net做HA，因为一个net可以add到多个dhcp-agent上

2、分担network节点的压力

3、部署dhcp-agent的同时，也要在该节点上部署metadata-agent

4、将net附加给某个dhcp-agent，可能需要命令手动


