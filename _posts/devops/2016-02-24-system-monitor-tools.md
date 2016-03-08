---
layout: post
title: 常用系统监控工具
modified:
categories: devops
excerpt: "常用系统监控工具教程"
tags: [系统监控]
comments: true
image:
  feature:
date: 2016-02-24T15:56:21+08:00
---

{% include _toc.html %}

## 网络监控类

### iftop

## IO监控类

### iostat

~~~ bash
$ iostat [ -c ] [ -d ] [ -N ] [ -k | -m ] [ -t ] [ -V ] [ -x ] [ -z ] [ device [...] | ALL ]
[ -p [ device [,...] | ALL ] ] [ interval [ count ] ]
~~~

* -c为汇报CPU的使用情况；
* -d为汇报磁盘的使用情况；
* -k表示每秒按kilobytes字节显示数据；
* -t为打印汇报的时间；
* -v表示打印出版本信息和用法；
* -x device指定要统计的设备名称，默认为所有的设备；
* interval指每次统计间隔的时间；
* count指按照这个时间间隔统计的次数。

常见用法：
iostat -d -k 1 5         查看磁盘吞吐量等信息。
iostat -d -x -k 1 5     查看磁盘使用率、响应时间等信息
iostat –x 1 5            查看cpu信息。
 
iostat -x 1（-x：显示扩展信息）
cpu：
%user：CPU处在用户模式下的时间百分比。
%nice：CPU处在带NICE值的用户模式下的时间百分比。
%system：CPU处在系统模式下的时间百分比。
%iowait：CPU等待输入输出完成时间的百分比。
%steal：管理程序维护另一个虚拟处理器时，虚拟CPU的无意识等待时间百分比。
%idle：CPU空闲时间百分比。

**注意** 
如果%iowait的值过高，表示硬盘存在I/O瓶颈，%idle值高，表示CPU较空闲，如果%idle值高但
系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量。%idle值如果持续低于10，那
么系统的CPU处理能力相对较低，表明系统中最需要解决的资源是CPU。
{: .notice}

disk：
rrqm/s:  每秒进行 merge 的读操作数目。即 rmerge/s
wrqm/s:  每秒进行 merge 的写操作数目。即 wmerge/s
r/s:  每秒完成的读 I/O 设备次数。即 rio/s
w/s:  每秒完成的写 I/O 设备次数。即 wio/s
rsec/s:  每秒读扇区数。即 rsect/s
wsec/s:  每秒写扇区数。即 wsect/s
rkB/s:  每秒读K字节数。是 rsect/s 的一半，因为每扇区大小为512字节。
wkB/s:  每秒写K字节数。是 wsect/s 的一半。
avgrq-sz:  平均每次设备I/O操作的数据大小 (扇区)。
avgqu-sz:  平均I/O队列长度。
await:  平均每次设备I/O操作的等待时间 (毫秒)。
svctm: 平均每次设备I/O操作的服务时间 (毫秒)。
%util:  一秒中有百分之多少的时间用于 I/O 操作，即被io消耗的cpu百分比

**注意** 
如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。
如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明
I/O 队列太长，io响应太慢，则需要进行必要优化。
如果avgqu-sz比较大，也表示有当量io在等待。
{: .notice}

## 磁盘IO测试

### fio

## 参考文档

* [Linux工具快速教程](http://linuxtools-rst.readthedocs.org/zh_CN/latest/index.html)



