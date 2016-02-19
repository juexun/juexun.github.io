---
layout: post
title: Libvirtd开启远程连接
modified:
categories: virt
excerpt: "Libvirtd 服务可以开启远程连接支持, 允许virsh等客户端通过TCP方式监控虚拟机"
tags: [Libvirt]
image:
  feature:
date: 2016-02-19T23:11:40+08:00
---

{% include _toc.html %}

## 0. 配置文件 

- /etc/libvirt/libvirtd.conf
- /etc/sysconfig/libvirtd

## 1. 开启远程访问支持

默认Libvirtd只支持本地unix socket访问，如果需要支持远程访问（默认16509端口），需要进行一下配置：

~~~ bash
$ sudo sed -i 's/#listen_tls = 0/listen_tls = 0/g' /etc/libvirt/libvirtd.conf
$ sudo sed -i 's/#listen_tcp = 1/listen_tcp = 1/g' /etc/libvirt/libvirtd.conf
$ sudo sed -i 's/#auth_tcp = "sasl"/auth_tcp = "none"/g' /etc/libvirt/libvirtd.conf
$ sudo sed -i 's/#LIBVIRTD_ARGS="--listen"/LIBVIRTD_ARGS="--listen"/g' /etc/sysconfig/libvirtd
~~~

## 2. 影响

目前已知,如果不开启远程连接支持,则无法进行虚拟机迁移,原因待查.