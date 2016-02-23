---
layout: post
title: Gluster 仲裁机制
modified:
categories: storage
excerpt: "Gluster 客户端仲裁和服务端仲裁机制对比"
tags: [Gluster]
comments: true
image:
  feature:
date: 2016-02-19T21:42:59+08:00
---

{% include _toc.html %}

[原文链接](http://dangzhiqiang.blog.51cto.com/7961271/1702214)

## 0. 概述

客户端仲裁只适用于副本卷，服务器仲裁适用于所有卷。

副本卷个数最好为奇数个，服务器端个数最好为不小于3的奇数。

## 1. 客户端仲裁

客户端仲裁适用于fuse、gfapi、nfs。

客户端仲裁功能是在AFR中继器中实现，所以只要该中继器被加载，就能提供客户端仲裁功能。仲裁检测自然由AFR中继器负责。如果检查失败，客户端则无法写入，返回EROFS错误。

正在进行写操作会因为板块（brick）数未达到仲裁要求的数量而失败，并且FOP会返回EROFS错误，后续的写操作会返回相同的错误，所以，这种情形下的错误会立即返回，没有等待超时机制。

仲裁状态是由客户端看到的活跃的（active）板块（brick）数决定的。具体的规则由quorum-type和quorum-count确定。

~~~ yaml
Option: cluster.quorum-type
Default Value: none
~~~

如果设置为“fixed”，只允许在quorum-count数量的板块（bricks）在线时写入。如果设置为“auto”，只允许超过一半的板块在线时写入，或者只允许一半的板块在第一次写入后继续写入。

~~~ yaml
Option: cluster.quorum-count
Default Value: (null)
~~~

如果quorum-type设置为“fixed”，只允许设置数量的板块（brick）在线时可以写入。如果quorum-type设置为其他值，设置的quorum-count值无效，会被覆盖。


## 2. 服务端仲裁

服务器端仲裁是由glusterd进程执行，但判断的是glusterfsd进程。

~~~ yaml
Option: cluster.server-quorum-type
Default Value: (null)
~~~

这个功能在服务器端实现，也就是在glusterd进程中。当glusterd检测到服务器端未达到法定人数时，就会停掉brick防止数据裂脑。当网络恢复达到法定人数时，就会恢复对应的brick。

~~~ yaml
Option: cluster.server-quorum-ratio
Default Value: (null)
~~~

仲裁成功的bricks可以继续写入，不成功的bricks会被设置成只读或者直接停掉，当再次仲裁成功上线后，会自动修复数据，因此可以防止裂脑，保证数据一致性性。

想深入了解服务器仲裁，可查看服务器仲裁测试方法：http://www.gluster.org/community/documentation/index.php/Features/Server-quorum

## 3. 取舍

* 客户端仲裁与服务器端仲裁那个好？
  
  如果启用服务器端仲裁，当出现裂脑情况时，仍然可以将数据写入卷中。服务器端仲裁为了更有效的避免和卷配置冲突，仲裁成为不可写入节点时，禁止执行volume set、peer probe等命令。如果要避免裂脑文件出现在卷中，最好使用客户端仲裁。
  
* 两种仲裁可以同时使用吗？有什么推荐配置？
  
  以我个人愚见，客户端仲裁就足够，但客户端仲裁只能用于复制卷环境。当然，两个仲裁机制可以同时使用。服务器端仲裁会直接干掉brick，干掉的brick甚至还允许进行读访问。