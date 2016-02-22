---
layout: post
title: 部署 Ceph 集群
modified:
categories: storage
excerpt: "Ceph基本概念及集群部署流程"
tags: [Ceph]
image:
  feature:
date: 2016-02-19T11:41:41+08:00
---

{% include _toc.html %}

Ceph 最初是一项关于存储系统的 PhD 研究项目，由 Sage Weil 在 University of California, SantaCruz（UCSC）实施。(话说，人家的博士研究项目质量真高啊，多少现在牛皮哄哄的开源项目刚开始也都是人家的一篇论文而已，膜拜ing)

可是，为啥这Ceph就这么火呢，据说很有料，能同时支持3种存储：

* 对象存储
* 文件存储
* 块设备存储

也就是说，它把人家几个一起才能干的事情都干了，这会儿你服了吧？！不服不行啊，这不，把我们领导都招来了。某天，领导找到我了，说：“诶，好像Ceph支持S3耶，琢磨着装一套来爽爽呗，反正你不是迟早要玩的嘛。” 好吧，我是说要玩啊，可没说现在就开始啊，唉！你是爽了，我可就惨了，555~~~

玩笑归玩笑，这正事还是得干，趁着这热乎劲，咱就开搞吧。

## 0. 资源篇

目前，网络上关于Ceph的资料还是不少的，大概整理如下：

* [Ceph官方文档](http://docs.ceph.com/docs/v0.80.5/)
* [Openfans的汉化版文档](http://docs.openfans.org/ceph)，相比官方文档，版本会低一点
* Ceph中国社区录制的[公开课](http://edu.51cto.com/course/course_id-4040.html)，视频教程，比较形象
* [Ceph中国社区](http://bbs.ceph.org.cn/explore/)
* 还有一些零碎的资料

根据我的学习经历，强烈推荐以官方文档为主，汉化文档为辅，再结合公开课视频，遇到问题配合上谷歌（查得严就用度娘），基本上搞定安装没多大问题。

我一开始偷懒，直接看汉化文档，照着步骤操作，结果踩坑无数；痛定思痛，转着投奔Ceph的官方文档。

## 1. 基础篇

通过阅读上面所列举的资料，我们可以知道，一个Ceph集群由以下几个组件组成：

### 1.0 OSD

​	这个组件负责数据管理，进行数据相关操作的；同时它还可以通过心跳机制检查其它OSD节点状态并通知给monitor。这是一个必选组件。

### 1.1 Monitor

​	这个组件负责维护集群状态图，这是一个必选组件。

### 1.2 MDS

​	这个组件负责保存Ceph 文件系统的元数据信息，这个组件仅当你需要使用Ceph文件系统的时候才需要安装，这是一个可选组件。

**一个Ceph集群至少由一个Mon和两个OSD组成**。（这里还有一点点问题，我在使用这种配置部署集群时，ceph health总是提示WARN，需要查证原因。）

关于Ceph对硬件和软件环境的要求，请参考官方文档，本文不再赘述。

## 2. 实践篇

动手之前，再提醒一次：**通读官方文档安装部分一遍**。本篇基于CentOS 6.x，以**快速**流程来完成Ceph存储集群的安装。

### 2.0 环境准备

#### 2.0.0 节点信息

| 角色                | 主机名   | IP          |
| :----------------- |:-----: | -----------: |
| Admin + Mon + OSD | admin | 192.168.1.2 |
| OSD               | node1 | 192.168.1.3 |
| OSD               | node2 | 192.168.1.4 |

#### 2.0.1 软件版本

* OS
  * CentOS 6.6
* Ceph
  * Hammer

#### 2.0.2 安装ceph-deploy

ceph-deploy是一套用来部署Ceph集群的工具。

* 配置YUM源
  
~~~ bash
$ sudo cat /etc/yum.repos.d/ceph.repo
[ceph-noarch]
name=Ceph noarch packages
baseurl=http://ceph.com/rpm-hammer/el6/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc
~~~
  
* 安装ceph-deploy
  
~~~ bash
$ sudo yum install ceph-deploy
~~~

#### 2.0.3 配置主机名

* 根据上表的信息，分别在三台机器的/etc/hosts文件中加入IP地址和主机名的映射

~~~ bash
$ sudo cat /etc/hosts
(前文略)
192.168.1.2 admin
192.168.1.3 node1
192.168.1.4 node2
~~~

* 设置三台服务器的主机名，确保主机名与上表一致。
  
  例如，在192.168.1.3上，执行hostname命令，结果如下所示：

~~~ bash
$ hostname -s
node1
~~~

* 确保在各台服务器上用主机名可以PING通集群内其它节点。

**在整个安装过程中，ceps-deploy使用主机短名进行通信，请务必保证/etc/hosts使用短名进行IP映射。**

#### 2.0.4 配置ceph用户

* 添加ceph用户
  
  * 登陆所有节点，添加ceph用户并设置密码
    
~~~ bash
$ sudo useradd -d /home/ceph -m ceph
$ sudo passwd ceph
~~~
    
  * 给ceph添加**root**权限
    
~~~ bash
$ echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph
$ sudo chmod 0440 /etc/sudoers.d/ceph
~~~
    
  * 关闭requiretty
    
    在CentOS下，sudo默认开启**requiretty**设置，会导致cep-deploy执行失败，必须关闭这个选项。
    
~~~ bash
$ sudo visudo
~~~
    
    找到**Default requiretty**,  修改为**Default:ceph !requiretty**。
  
* 配置admin节点到所有其它节点免密码登陆
  
~~~ bash
$ ssh ceph@admin
$ ssh-keygen
$ ssh-copy-id ceph@admin (因为admin同时也是osd和mon)
$ ssh-copy-id ceph@node1
$ ssh-copy-id ceph@node2
~~~
  
* 配置配置admin节点ceph用户目录下的**".ssh/config"**文件，确保cep-deploy登陆cep节点时使用ceph用户
  
~~~ text 
Host node1
 Hostname node1
 User ceph
Host node2
 Hostname node2
 User ceph
Host admin
 Hostname admin
 User ceph
~~~

环境准备工作到此结束，下面开始部署集群。

### 2.1 部署集群

开始之前，请使用ceph用户登陆admin节点，以下所有操作都将以ceph用户进行。

#### 2.1.0 重置环境

如果安装过程中发生错误，可进行如下操作：

* 重置配置
  
~~~ bash
$ ceph-deploy purgedata <主机名>
$ ceph-deploy forgetkeys
~~~
  
* 重置配置并重新安装软件
  
~~~ bash
$ ceph-deploy purge <主机名>
~~~
  
  ​

#### 2.1.1 创建安装目录

创建一个安装目录，在该目录下进行安装操作，安装过程中产生的日志和配置文件都会保存在该目录下。

~~~ bash
ssh ceph@admin
$ mkdir my-cluster
$ cd my-cluster
~~~

#### 2.1.2 创建集群

创建一个集群，并修改集群的默认配置文件。

~~~ bash
$ ceph-deploy new <mon节点主机名1> <mon节点主机名2>
~~~

在本文中，我们只有一个mon节点，名字为admin，所以命令如下：

~~~ bash
$ ceph-deploy new admin
~~~

以上命令没有指定集群名子，所以集群的名字为默认值"ceph"。命令结束后，在当前目录下生成名为“ceph.conf”的配置文件。

~~~ bash
$ cat ceph.conf
[global]
fsid = c6070eba-cf45-4cb8-baee-f0aee34d62ff
mon_initial_members = admin
mon_host = 192.168.1.2
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
filestore_xattr_use_omap = true
~~~

我们加入以下两行：

~~~ text 
osd_pool_default_size = 2
public_network = 192.168.1.0/22
~~~

**osd_pool_default_size**：默认副本数

**public_network**：管理网络

**cluster_network**: 数据网络，节点间进行数据传输的网络，不设置则使用管理网络进行数据传输

#### 2.1.3 安装节点

安装各个节点上的组件。

（安装之前，先要把admin上的/etc/yum.repos.d/ceph.repo配置删除。因为ceps-deploy在安装ceph-replease时，会生成cep.repo。如果这个文件已经存储，则无法覆盖，会导致后续软件无法安装。）

~~~ bash
$ ceph-deploy install admin node1 node2
~~~

#### 2.1.4 初始化mon

初始化mon节点，并收集keys。

~~~ bash
$ ceph-deploy mon create-initial
~~~

该命令完成后，会在当前目录下生成以下文件：

~~~ text
{cluster-name}.client.admin.keyring
{cluster-name}.bootstrap-osd.keyring
{cluster-name}.bootstrap-mds.keyring
~~~

#### 2.1.5 添加OSD

* 登录所有osd节点
  
~~~ bash
ssh ceph@osd-node
$ sudo mkdir /var/lib/osd
~~~
  
* 登录admin节点，准备OSD
  
~~~ bash
ssh ceph@admin
$ ceph-deploy osd prepare admin:/var/lib/osd node1:/var/lib/osd node2:/var/lib/osd
~~~
  
* 激活OSD
  
~~~ 
$ ceph-deploy osd activate admin:/var/lib/osd node1:/var/lib/osd node2:/var/lib/osd
~~~

#### 2.1.6 配置ceph CLI

~~~ bash
$ ceph-deploy install admin node1 node2
~~~

完成此步骤后，在集群的任何节点上，都可以使用ceph命令查看集群的相关信息，而不需要知道mon地址和key。

使用ceph命令之前，确保**“/etc/ceph/ceph.client.admin.keyring”**有读权限。

~~~ bash
sudo chmod +r /etc/ceph/ceph.client.admin.keyring
~~~

#### 2.1.7 查看集群状态

~~~ bash
$ ceph health
HEALTH_OK
~~~

### 2.2 注意事项

* 关于Ceph源的问题
  
  从国内直接使用Ceph官网源安装节点组件时，时间比较长，会导致安装失败。建议将网络源下载做成本地源，然后修改ceph.repo文件进行安装。


* 软件重置时，如果需要完成重置，请务必删除/var/lib/ceph目录下的文件
  
* 关于重复安装的问题
  
  由于刚开始玩Ceph，安装过程难免问题多多，免不了重复安装。为了节省时间，我把Ceph官方源下载到本地，并制作了本地YUM源。每次重新安装之前，必须清除安装环境和配置。另外，ceph-deploy工具每次安装或卸载时，都会先安装或者卸载ceph-release，这个包其实就是生成官方YUM源配置文件。由于我使用的是本地源，所以这个包不需要安装。所以我修改了ceph-deploy，去掉了对ceph-release的安装和卸载操作。
  
* public_network或cluster_network至少必需配置一项

