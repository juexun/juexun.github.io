---
layout: post
title: 部署 Mariadb Galera Cluster
modified:
categories: database
excerpt: "基于CentOS 6.x/7.x 部署 MariaDB Galera Cluster"
tags: [MariaDB]
image:
  feature:
date: 2016-02-19T09:56:52+08:00
---

{% include _toc.html %}

## 0. 前言

MariaDB是MySQL的一个分支，选择MariaDB的一个最初的动机是：

- 简单的集群配置
- 能较好的兼容MySQL

## 1. 安装环境

配置Galera集群建议至少用3个节点（如果只有两个节点，需要添加仲裁节点，本文没有相关说明）。

| 操作系统       | IP         | 主机名     | 集群名称     |
| :--------- | :------------:| :---------: | -----------: |
| CentOS 6.6 | 192.168.3.100 | dbserver0 | vmmscluster |
| CentOS 6.6 | 192.168.3.101 | dbserver1 | vmmscluster |
| CentOS 6.6 | 192.168.3.102 | dbserver2 | vmmscluster |
{: rules="groups"}

## 2. 软件安装与配置

本章节所有操作在所有节点上无差异执行。

### 2.0 YUM源配置

* 官方源
  
~~~ bash
$ sudo cat /etc/yum.repos.d/mariadb.repo
[mariadb]
name=MariaDB
baseurl=http://yum.mariadb.org/10.0/centos6-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
~~~
  
* 本地源
  
  为了安装方便，我们也可以从官方把RPM下载到本地，并制作本地YUM源
  
~~~ bash
$ sudo cat /etc/yum.repos.d/mariadb.repo
[mariadb]
name=MariaDB
baseurl=<本地源路径>
enabled=1
gpgcheck=0
~~~

### 2.1 安装服务器

* 安装之前
  
  在安装MariaDB软件之前，必须卸载所有已安装的MySQL的相关软件包，以及其它不兼容版本的MariaDB软件包。
  
~~~ bash
$ sudo yum erase *mysql* MariaDB* galera
~~~
  
* 安装服务器软件
  
~~~ bash
$ sudo yum install MariaDB-Galera-server MariaDB-client galera
~~~

### 2.2 配置Galera集群

#### 2.2.0 配置第一个节点

本节所有操作在dbserver0上进行，且仅在该节点上进行

* 配置root用户密码
  
~~~ bash
(安装完成后，数据库服务不会自动启动)
$ sudo /etc/init.d/mysql start
$ sudo mysql_secure_installation
- 设置root密码
- 删除匿名访问
- 删除测试数据库
- 刷新权限表
$ sudo /etc/init.d/mysql stop
~~~
  
* 配置文件

~~~ bash
$ sudo cat /etc/my.cnf.d/galera.cnf
[galera]

wsrep_provider=/usr/lib64/galera/libgalera_smm.so
binlog_format=ROW

wsrep_cluster_address=gcomm://dbserver0,dbserver1,dbserver2

wsrep_node_address='192.168.3.101'
wsrep_node_name='dbserver1'

wsrep_cluster_name='vmmscluster'
~~~

#### 2.2.1 配置其它节点

将上一步中生成的 ‘/etc/my.cnf.d/galera.cnf’ 文件拷贝到其它节点的相应路径，并根据实际信息修改一下字段即可

* wsrep_node_name: 节点主机名
* wsrep_node_address: 节点IP地址

## 3 集群管理

### 3.0 启动集群

* 启动第一个节点
  
~~~ bash
$ sudo /etc/init.d/mysql bootstrap
~~~
  
* 启动其它节点
  
~~~ bash
$ sudo /etc/init.d/mysql start
~~~


* 当集群中有任何一个节点在运行时，节点重启或者加入新节点都只需要直接启动即可
  
~~~ bash
$ sudo /etc/init.d/mysql start
~~~


* 查看集群状态
  
~~~ bash
$ mysql -u root -p -e "show status like 'wsrep%'"
~~~
  
  关注几个关键的参数：
  
  | 参数       | 状态         | 功能   |
  | :---------| :------     :| -----------: |
  | wsrep_connected | on | 链接已开启 |
  | wsrep_local_index | 1 | 在集群中的索引值 |
  | wsrep_cluster_size | 3 | 集群中节点的数量 |
  | wsrep_incoming_addresses |  | 集群中节点的访问地址 |
  {: rules="groups"}
  
## 4 参考文档

* [MariaDB Galera Cluster 部署](http://code.oneapm.com/database/2015/07/02/mariadb-galera-cluster/)
  
  基于CentOS 7部署MariaDB Galera Cluster