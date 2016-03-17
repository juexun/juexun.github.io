---
layout: post
title: 计算节点配置指南
modified:
categories: iaas
excerpt: "基于CentOS7的计算节点部署流程"
tags: [自研]
comments: true
image:
  feature:
date: 2016-02-19T22:31:00+08:00
---

{% include _toc.html %}

## 0. 安装CentOS7

### 0.0 使用Minimal模式安装

### 0.1 基本网络配置

* 关闭zeroconf
  
~~~ bash
$ sudo cat /etc/sysconfig/network | grep NOZEROCONF
NOZEROCONF=yes
~~~

* 关闭NetworkManager服务
  
~~~ bash 
$ sudo systemctl stop NetworkManager.service 
$ sudo systemctl disable NetworkManager.service
$ sudo systemctl status NetworkManager.service  
NetworkManager.service - Network Manager
 Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; disabled; vendor preset: enabled)
 Active: inactive (dead) since 一 2016-02-01 18:05:42 CST; 18s ago
~~~
  
* 所有主机必须使用相同的网络接口名称,如eth*, em*等
* 为了增强网络的可靠性,尽量配置多网卡绑定,多主机使用相同的绑定接口名称,如bond0等
  
**注意:** 如果做了多网卡绑定,对原始的网络接口名称不做强制要求
{: .notice}
  
* 尽可能区分存储网络,管理网络和业务网络,如果无法分三个网络,至少保证存储网络独立

### 0.2 VLAN配置

* 加载8021Q内核模块
  
~~~ bash
$ sudo modprobe 8021q
$ sudo lsmod | grep 8021q
8021q                  29022  0 
garp                   14384  1 8021q
mrp                    18542  1 8021q
~~~

* 配置网络接口

  > 以下实例中，VLAN Tag 为16，物理网卡为em1；另外，当多个网络同时存在时，只能一个网卡设置GATEWAY属性。
  
~~~ bash
$ cat /etc/sysconfig/network-scripts/ifcfg-em1.16
VLAN=yes
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
DEVICE=em1.16
IPADDR=172.16.0.10
PREFIX=16
~~~

### 0.3 系统安全设置

* 配置防火墙服务
  
**注意:** 暂时关闭防火墙,清空防火墙规则,待定义出所有需要的防火墙规则后再开启
{: .notice}

~~~ bash 
$ sudo iptables -L -n
$ sudo iptables -P INPUT ACCEPT
$ sudo iptables -F
$ sudo iptables -X
~~~
  
* 关闭SELinux
  
**注意:** 暂时关闭SELinux,待定义出所有所需的规则后再开启
{: .notice}

  将SELINUX设置为disabled,如下所示
  
~~~ bash 
$ sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
~~~

### 0.4 配置主机名

* 在/etc/hosts文件中关联本机IP和主机名

## 1. 配置虚拟化环境

### 1.0 安装依赖包

* 安装虚拟化组件
  
~~~ bash
$ sudo yum groupinstall 'Virtualization Host'
~~~

### 1.1 配置libvirtd

* /etc/libvirt/libvirtd.conf
  
~~~ bash
$ sudo sed -i 's/#listen_tls = 0/listen_tls = 0/g' /etc/libvirt/libvirtd.conf
$ sudo sed -i 's/#listen_tcp = 1/listen_tcp = 1/g' /etc/libvirt/libvirtd.conf
$ sudo sed -i 's/#auth_tcp = "sasl"/auth_tcp = "none"/g' /etc/libvirt/libvirtd.conf
~~~
  
* /etc/sysconfig/libvirtd
  
~~~ bash
$ sudo sed -i 's/#LIBVIRTD_ARGS="--listen"/LIBVIRTD_ARGS="--listen"/g' /etc/sysconfig/libvirtd
~~~

### 1.2 配置qemu

* /etc/libvirt/qemu.conf
  
  > 关于QEMU的配置,还有很多优化的空间,需要深入的研究
  
~~~ bash
$ sudo sed -i 's/#user = "root"/user = "root"/g' /etc/libvirt/qemu.conf
$ sudo sed -i 's/#group = "root"/group = "root"/g' /etc/libvirt/qemu.conf
~~~

### 1.3 启动libvirtd服务

* 启动libvirtd
  
~~~ bash
$ sudo systemctl start libvirtd
~~~
  
* 确认libvirtd服务是否被设置为开机自启动
  
~~~ bash
$ sudo systemctl is-enabled libvirtd.service
enabled
~~~
  
* 将libvirtd服务设置为开机自启动
  
  > 如果上一步返回的结果是enabled,则跳过此步
  
~~~ bash
$ sudo systemctl enable libvirtd.service
~~~

### 1.4 libvirt-guests服务

详情参考 "[libvirt配置](http://juexun.github.io/virt/libvirt-config/)"

### 1.5 配置dnsmasq服务

libvirtd服务配置的默认虚拟网桥使用dnsmasq作为DHCP服务为虚拟机分配IP,SunrunIaaS中也使用了dnsmasq来分配IP.如果同时使用会引起冲突,所以需要将默认虚拟网桥删除,并关闭dnsmasq服务.

* 删除默认虚拟网桥(virbr0)定义
  
查看default网桥是否存在
    
~~~ bash
$  ifconfig virbr0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
(以下略)

$ sudo virsh net-list | grep default
default              active     yes           yes
~~~
  
* 删除default网桥
  
**注意:** 如果上一步中,没有virbr0, 且default网桥不存在,则跳过这一步
{: .notice}
  
~~~ bash
$ sudo virsh net-destroy default
Network default destroyed
$ sudo virsh net-undefine default
Network default has been undefined
~~~
  
* 确认dnsmasq服务不会开机自启动
  
  * 查询dnsmasq服务状态
    
~~~ bash
$ sudo systemctl is-enabled dnsmasq.service
disabled
~~~
    
  * 关闭dnsmasq服务开机自启动
    
**注意:** 如果上一步为disable,则跳过这一步
{: .notice}
    
~~~ bash
$ sudo systemctl disable dnsmasq.service
~~~

### 1.6 重启服务器

  为了确保配置有效,完成以上配置后,请务必重启服务器.

  服务器重启后,检查以下项目:

* zeroconfig 已经关闭
  
~~~ bash
$ route -n |grep 169.254.0.0
(输出为空)
~~~


* libvirtd 服务开机自启动,且已经正在运行
  
~~~ bash
$ sudo systemctl status libvirtd
● libvirtd.service - Virtualization daemon
Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
Active: active (running) since Sun 2016-01-31 17:47:56 CST; 1min 10s ago
(以下略)
~~~
  
* 无virbr0 网络接口
  
~~~ bash
$ ifconfig virbr0
virbr0: error fetching interface information: Device not found
~~~
  
* 没有 default 虚拟网桥
  
~~~ bash
$ sudo virsh net-list | grep default
(此处无任何输出)
~~~
  
* dnsmasq 服务未开机自启动,且未启动
  
~~~ bash
$ sudo systemctl status dnsmasq
● dnsmasq.service - DNS caching server.
Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; disabled; vendor preset: disabled)
Active: inactive (dead)
~~~
  
* SELinux已关闭
  
**注意:** 当前设置为关闭SELinux,如果开启防火墙设置,此处会不同
{: .notice}
  
~~~ bash
$ sudo sestatus
SELinux status:                 disabled
~~~
  
* 防火墙(firewalld) 服务未开机自启动,且当前未启动
  
**注意:** 当前设置为关闭防火墙,如果开启防火墙设置,此处会不同
{: .notice}
  
~~~ bash
$ sudo systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)

Active: inactive (dead)
~~~
  
* 防火墙规则为空
  
**注意:** 当前设置为关闭防火墙,如果开启防火墙设置,此处会不同
{: .notice}
  
~~~ bash
$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
~~~
