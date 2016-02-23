---
layout: post
title: 虚拟机性能优化
modified:
comments: true
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

## 内存优化

### KSM

在CentOS下KSM是打开的，Debian下KSM是关闭的。KSM的原理，是多个进程中，Linux将内核相似的内存页合并成一个内存页。这个特性，被KVM用来减少多个相似的虚拟机的内存占用，提高内存的使用效率。由于内存是共享的，所以多个虚拟机使用的内存减少了。这个特性，对于虚拟机使用相同镜像和操作系统时，效果更加明显。

但是，事情总是有代价的，使用这个特性，都要增加内核开销，用时间换空间。所以为了提高效率，可以将这个特性关闭。

* 关闭方法

~~~ bash
$ sudo echo 0 > /sys/kernel/mm/ksm/run
或者
chkconfig ksm off
chkconfig ksmtuned off
~~~

### Huge Page

Intel 的X86 CPU通常使用4Kb内存页，当是经过配置，也能够使用巨页(huge page):

* 4MB on x86_32
* 2MB on x86_64 and x86_32 PAE

使用巨页，KVM的虚拟机的页表将使用更少的内存，并且将提高CPU的效率。最高情况下，可以提高20%的效率！

* 开启方法

~~~ bash
$ sudo mount -t hugetlbfs hugetlbfs /dev/hugepages
(保留一些内存给巨页)
$ sudo sysctl vm.nr_hugepages=516
~~~

* 虚拟机相关配置

~~~ xml
<memoryBacking>
<hugepages/>
</memoryBacking>
~~~

## 参考文档

* [淘宝子团关于KVM调优的分享](http://www.pubyun.com/blog/openstack/%E6%B7%98%E5%AE%9D%E5%AD%90%E5%9B%A2%E5%85%B3%E4%BA%8Ekvm-%E8%B0%83%E4%BC%98%E7%9A%84%E5%88%86%E4%BA%AB/)

* [CentOS6 KSM特性](http://www.linuxtopia.org/online_books/rhel6/rhel_6_virtualization/rhel_6_virtualization_chap-KSM.html)

