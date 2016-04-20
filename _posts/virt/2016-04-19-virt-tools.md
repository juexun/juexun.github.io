---
layout: post
title: Guestfish套件
modified:
categories: virt
excerpt: "KVM虚拟化相关工具"
tags: []
image:
  feature:
date: 2016-04-19T08:49:22+08:00
---

## 安装

~~~ bash
$ sudo yum install libguestfs-tools
~~~

**注意** 默认安装不支持windows系统,如果需要增加Windows系统的支持,需要安装以下工具包
{: .notice}

~~~ bash
$ sudo yum install libguestfs-winsupport
~~~

## 镜像相关工具

###  virt-inspect
