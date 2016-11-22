.. niusmallnan documentation master file, created by
   sphinx-quickstart on Tue Feb 18 13:49:43 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


=======================================
租户用户体系设计方案
=======================================
东网云平台租户用户体系设计方案


设计概览 
================
我们设计的最终目标是这样的:

.. image:: /images/tenant_user_system.png

补充说明:

- 上图中不同的颜色代表所属不同的角色，不同的角色意味着不同的操作权限
- 租户是用户的集合，租户本身不具有用户特性
- 每个租户下至少有一个AdminUser，用于管理这个租户及其麾下用户
- 一个用户只能属于一个租户


各个表的概念设计:

.. image:: /images/tenant_user_dbeer.png









