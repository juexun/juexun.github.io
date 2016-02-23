---
layout: post
title: RabbitMQ 安装与配置
modified:
categories: middleware
excerpt: "基于CentOS 6.x的RabbitMQ服务的安装和配置"
tags: [RabbitMQ]
image:
  feature:
date: 2016-02-19T23:15:07+08:00
---

{% include _toc.html %}

## 0. 安装RabbitMQ

### 0.0 使用EPEL源安装

CentOS EPEL源包含了RabbitMQ的相关软件包，配置好YUM源后，我们可以直接安装：

~~~ bash
$ sudo yum install rabbitmq-server
~~~

### 0.1 基于Docker安装

待续

## 1. 服务控制

* 启动服务
  
~~~ bash
$ sudo /etc/init.d/rabbitmq-server start
~~~


* 停止服务
  
~~~ bash
$ sudo /etc/init.d/rabbitmq-server stop
~~~


* 设置开机启动
  
~~~ bash
$ sudo chkconfig rabbitmq-server on
~~~


* 查看服务状态
  
~~~ bash
$ sudo rabbitmqctl status
~~~


* 服务端口

|---
| 端口号        | 功能                            |
|:-------------|:--------------------------------|
| 4369         |                                 |
|---
| 5672         | AMQP 0-9-1 without TLS          |
|---
| 5671         | AMQP 0-9-1 with TLS             |
| 15672        | if management plugin is enabled |
| 61613, 61614 | if STOMP is enabled             |
| 1883, 8883   | if MQTT is enabled              |
|---

## 2. 系统配置

### 2.0 软件目录

| 路径                               | 功能                              |
| :-------------------------------- | :------------------------------- |
| /etc/rabbitmq                    | 配置文件目录                          |
| /usr/lib/rabbitmq/lib            | RabbitMQ相关库文件                   |
| /usr/lib/rabbitmq/bin            | RabbitMQ相关命令文件路径，可设置到用户PATH变量中。 |
| /usr/sbin/rabbitmq-server        | RabbitMQ服务器程序                   |
| /usr/sbin/rabbitmqctl            | RabbitMQ控制程序                    |
| /etc/rc.d/init.d/rabbitmq-server | RabbitMQ服务启动脚本                  |
| 其它                               | 文档、手册及日志循环配置文件                  |

### 2.1命令说明

| 命令               | 功能                |
| :---------------- | :----------------- |
| rabbitmq-plugins | RabbitMQ服务器插件管理工具 |
| rabbitmqctl      | RabbitMQ服务控制工具    |

### 2.2 插件管理

#### 2.2.0 插件列表

~~~ bash
$ rabbitmq-plugins list
~~~

| 名称                                | 功能            |
| :--------------------------------- | :-------------: |
| amqp_client                       |               |
| cowboy                            |               |
| eldap                             |               |
| mochiweb                          |               |
| rabbitmq_amqp1_0                  |               |
| rabbitmq_auth_backend_ldap        |               |
| rabbitmq_auth_mechanism_ssl       |               |
| rabbitmq_consistent_hash_exchange |               |
| rabbitmq_federation               |               |
| rabbitmq_federation_management    |               |
| rabbitmq_jsonrpc                  |               |
| rabbitmq_jsonrpc_channel          |               |
| rabbitmq_jsonrpc_channel_examples |               |
| rabbitmq_management               | RabbitMQ管理控制台 |
| rabbitmq_management_agent         |               |
| rabbitmq_management_visualiser    |               |
| rabbitmq_mqtt                     |               |
| rabbitmq_shovel                   |               |
| rabbitmq_shovel_management        |               |
| rabbitmq_stomp                    |               |
| rabbitmq_tracing                  |               |
| rabbitmq_web_dispatch             |               |
| rabbitmq_web_stomp                |               |
| rabbitmq_web_stomp_examples       |               |
| rfc4627_jsonrpc                   |               |
| sockjs                            |               |
| webmachine                        |               |

#### 2.2.1 插件控制

* 启用插件
  
~~~ bash
$ sudo rabbitmq-plugins enable <插件名>
~~~
  
* 禁用插件
  
~~~ bash
$ sudo rabbitmq-plugins disable <插件名>
~~~

### 2.3 用户管理

* 默认用户

| 用户名 | 密码    |
| :----- | :----- |
| guest | guest |

* 用户列表
  
~~~ bash
$ sudo rabbitmqctl list_users
~~~

* 新增用户
  
~~~ bash
$ sudo rabbitmqctl add_user <用户名> <密码>
~~~

* 删除用户
  
~~~ bash
$ sudo rabbitmqctl delete_user <用户名>
~~~

* 修改密码
  
~~~ bash
$ sudo rabbitmqctl change_password <用户名> <新密码>
~~~

* 清空密码
  
~~~ bash
$ sudo rabbitmqctl clear_password <用户名>
~~~

* 设置标签
  
~~~ bash
$ sudo rabbitmqctl set_user_tags <用户名> <标签>
~~~

* 清空标签
  
~~~ bash
$ sudo rabbitmqctl set_user_tags <用户名>
~~~

### 2.4 访问控制

#### 2.4.1 虚拟主机(vhost)

* 增加虚拟主机
  
~~~ bash
$ sudo rabbitmqctl add_vhost <主机名>
~~~

* 删除虚拟主机
  
~~~ bash
$ sudo rabbitmqctl delete_vhost <主机名>
~~~

* 虚拟主机列表
  
~~~ bash
$ sudo rabbitmqctl list_vhosts
~~~

#### 2.4.2 权限(permission)

* 设置虚拟主机的某个用户权限
  
~~~ bash
$ sudo rabbitmqctl set_permissions [-p vhostpath] {user} {conf} {write} {read}
~~~

* 清空虚拟主机的某个用户权限
  
~~~ bash
$ sudo rabbitmqctl clear_permissions [-p vhostpath] {username}
~~~

* 虚拟主机所有用户权限列表
  
~~~ bash
$ sudo rabbitmqctl list_permissions [-p vhostpath]
~~~

* 某个用户的权限列表
  
~~~ bash
$ sudo rabbitmqctl list_user_permissions {username}
~~~

### 2.5 策略管理（policy）

待续

## 3 服务器运行状态

待续

## 4 其它