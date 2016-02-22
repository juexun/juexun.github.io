---
layout: post
title: 虚拟机优化性能
modified:
categories: virt
excerpt: "虚拟机各方面性能优化相关的配置"
tags: [KVM]
image:
  feature:
date: 2016-02-22T21:56:48+08:00
---

{% include _toc.html %}

## 磁盘IO优化

* cachemode

writeback: 模式速度最快,但是可能有丢数据的风险

## 网络优化

### CentOS7支持Virtio网卡多队列

* 虚拟机的xml网卡配置

~~~ xml
<interface type=<网络类型>>
<source network=<网桥名称> />
<model type='virtio'/>
<driver name='vhost' queues='N'/>
<!-- N:1 - 8 最多支持8个队列 -->
</interface>
~~~

* 在虚拟机上执行以下命令开启多队列网卡

~~~ bash
$ sudo ethtool -L eth0 combined M
(M 1 - N M小于等于N)
~~~

