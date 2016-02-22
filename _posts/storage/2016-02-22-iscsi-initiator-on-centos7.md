---
layout: post
title: iSCSI 客户端操作
modified:
categories: storage
excerpt: "在CentOS7上部署iSCSI客户端,及其它常规操作"
tags: [iSCSI]
image:
  feature:
date: 2016-02-22T17:27:36+08:00
---

{% include _toc.html %}

## 配置iSCSI Initiator

* 安装客户端软件

~~~ bash
$ sudo yum -y install iscsi-initiator-utils
~~~

* 查看本机IQN

~~~ bash
$ sudo cat /etc/iscsi/initiatorname.iscsi
InitiatorName=<iqn........>
~~~

* 配置用户名和密码

** 注: ** 如果对安全没有特别要求,这一步可跳过
{: .notice}

~~~ bash
$ sudo cat /etc/iscsi/iscsid.conf
(忽略无关内容)

node.session.auth.authmethod = CHAP

node.session.auth.username = <用户名>
node.session.auth.password = <密码>

~~~

* 发现iSCSI Target

~~~ bash
$ sudo iscsiadm -m discovery -t sendtargets -p <iSCSI Target IP>
(以下列出可连接的Target IQN,格式如下)
192.168.0.110:3260,1 iqn.2003-10.com.lefthandnetworks:iaas:80:backup
~~~

* 登录Target

~~~ bash
$ sudo iscsiadm -m node --login 
~~~

* 查看已连接的会话

~~~ bash
$ sudo iscsiadm -m session -o show 
~~~

至此,客户端已经连接Target,可作为一个磁盘使用.

## 磁盘分区及格式化

* 查看磁盘

~~~ bash
$ sudo fdisk -l
此处可看到新增的磁盘信息,例如sdb
~~~

* 分区

~~~ bash
$ sudo parted --script /dev/sdb "mklabel msdos"
$ sudo parted --script /dev/sdb "mkpart primary 0% 100%" 
~~~

* 格式化

~~~ bash
$ sudo mkfs.xfs -i size=1024 -s size=4096 /dev/sdb1
~~~

* 挂载

~~~ bash
$ sudo mount /dev/sdb1 /mnt
~~~

