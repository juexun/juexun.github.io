---
layout: post
title: Libvirt 配置
modified:
comments: true
categories: virt
excerpt: "Libvirtd 服务可以开启远程连接支持, 允许virsh等客户端通过TCP方式监控虚拟机"
tags: [Libvirt]
image:
  feature:
date: 2016-02-19T23:11:40+08:00
---

{% include _toc.html %}

## 0. libvirtd 服务

### 0.0 配置文件 

- /etc/libvirt/libvirtd.conf
- /etc/sysconfig/libvirtd

### 0.1 开启远程访问支持

* 配置

默认Libvirtd只支持本地unix socket访问，如果需要支持远程访问（默认16509端口），需要进行一下配置：

~~~ bash
$ sudo sed -i 's/#listen_tls = 0/listen_tls = 0/g' /etc/libvirt/libvirtd.conf
$ sudo sed -i 's/#listen_tcp = 1/listen_tcp = 1/g' /etc/libvirt/libvirtd.conf
$ sudo sed -i 's/#auth_tcp = "sasl"/auth_tcp = "none"/g' /etc/libvirt/libvirtd.conf
$ sudo sed -i 's/#LIBVIRTD_ARGS="--listen"/LIBVIRTD_ARGS="--listen"/g' /etc/sysconfig/libvirtd
~~~

* 影响

目前已知,如果不开启远程连接支持,则无法进行虚拟机迁移,原因待查.

## 1. libvirt-guests 服务

### 0.0 配置文件

- /etc/sysconfig/libvirt-guests

### 0.1 配置项

| 配置项 | 默认值 | 功能 |  说明  
| :- | :-| :- | :-
| URIS | default | libvirtd服务的URL | 不需要设置,默认即可
| ON_BOOT | ignore | 宿主机启动时开启所有关机之前正在运行的虚拟机 | 不管autostart是否被设置
| START_DELAY | 0 | 虚拟机的启动间隔 | 0表示虚拟机可以同时启动,没有间隔
| ON_SHUTDOWN | suspend | 宿主机关机时,对虚拟机进行的操作 | 可以被设置为suspend和shutdown;当设置为shutdown时,必须设置SHUTDOWN_TIMEOUT
| SHUTDOWN_TIMEOUT | 300 | 虚拟机关闭的超时时间 | 以秒为单位, 设置为0时,则不等待
| PARALLEL_SHUTDOWN | | 可以同时关闭的虚拟机数目 | 设置为0时,不允许虚拟机同时关闭
| BYPASS_CACHE | | Restore 虚拟机时,by-pass 文件系统缓存.| 0为关闭; 1为开启. 在一些文件系统下, 会使操作变慢.

### 0.2 开启libvirt-guests服务

~~~ bash
$ sudo systemctl start libvirt-guests
~~~

## 参考文档

* [libvirt-guests 配置官方文档](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Virtualization_Administration_Guide/sub-sect-Shutting_down_rebooting_and_force_shutdown_of_a_guest_virtual_machine-Manipulating_the_libvirt_guests_configuration_settings.html)