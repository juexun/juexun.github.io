---
layout: post
title: 部署Global Filesytem v2
modified:
categories: storage
excerpt: "基于iSCSI存储,部署GFS2集群文件系统"
tags: [GFS2]
image:
  feature:
date: 2016-02-19T21:33:21+08:00
---

{% include _toc.html %}

**注:** 本文所述流程仅在CentOS 6.x版本验证通过
{: .notice}

## 1. 部署

以下操作如非特别注明，则需要在所有节点上执行。

### 1.0 安装环境

|  主机名  |      IP       |     OS     |      硬件      |
| :---: | :-----------: | :--------: | ----------: |
| node0 | 192.168.3.100 | CentOS 6.x |    PC 服务器    |
| node1 | 192.168.3.101 | CentOS 6.x |    PC 服务器    |
|   -   | 192.168.0.110 |     -      | HP P4000 SAN |

### 1.1 安装集群

#### 1.1.0 配置主机名

编辑hosts文件，加入主机名和IP的映射。

~~~ bash
$ sudo cat /etc/hosts
127.0.0.1   localhost localhost.localdomain
::1         localhost localhost.localdomain 
192.168.3.100 node0
192.168.3.101 node1
~~~

#### 1.1.1 安装相关软件

~~~ bash
$ sudo yum install cman openais gfs* lvm2* rgmanager system-config-cluster scsi-target-utils cluster-snmp
$ sudo /etc/init.d/iptables stop
(关闭防火墙确保实验环境通信顺畅，具体在开启防火墙的情况下需要开放的端口仍需后续验证)
(同时，确保selinux处于关闭状态)
~~~

#### 1.1.2 配置lvm

* 将locking_type = 1，改为locking_type = 3，允启用同时读写
* 设置fallback_to_local_locking=0，以禁止回写，避免导致裂脑

#### 1.1.3 配置集群

* 编辑/etc/cluster/cluster.conf，增加下列内容：
  
~~~ xml 
<?xml version="1.0"?>
<cluster config_version="2" name="gfs_cluster">
<fence_daemon post_fail_delay="0" post_join_delay="3"/>
<clusternodes>
<clusternode name="node0" nodeid="1" votes="1">
<fence>
<method name="1">
<device name="manual_fence" nodename="node0"/>
</method>
</fence>
</clusternode>
<clusternode name="node1" nodeid="2" votes="2">
<fence>
<method name="1">
<device name="manual_fence" nodename="node1"/>
</method>
</fence>
</clusternode>
</clusternodes>
<cman/>
<fencedevices>
<fencedevice agent="fence_manual" name="manual_fence"/>
</fencedevices>
<rm>
<failoverdomains/>
<resources/>
</rm>
</cluster>
~~~


* 验证配置文件是否合法
  
~~~ bash
$ sudo ccs_config_validate
~~~

#### 1.1.4 启动集群

~~~ bash
$ sudo service cman start
$ sudo service clvmd start
$ sudo service rgmanager start
$ sudo clusterat
Cluster Status for gfs_cluster @ Wed Jan 27 21:39:17 2016
Member Status: Quorate
Member Name                                                     ID   Status
------ ----                                                     ---- ------
node0                                                           1 Online, Local
node1                                                           2 Online
~~~

### 1.2 配置卷

#### 1.2.1 加载iSCSI LUN

以下操作在标识为‘Local’的节点上进行

~~~ bash
$ sudo iscsiadm -m discovery -t sendtargets -p 192.168.0.110
$ sudo iscsiadm -m node -T <iSCSI的IQN> -p 192.168.0.110 --login
$ sudo fdisk -l
(查看iSCSI LUN的设备路径)
$ sudo pvcreate /dev/sdd -----------<iSCSI LUN的设备路径, 本文为>
$ sudo vgcreate vg_iscsi /dev/sdd
$ sudo pvs
  PV         VG       Fmt  Attr PSize   PFree 
  /dev/sdd   vg_iscsi lvm2 a--  690.00g 690.00g
$ sudo lvcreate -L 680g -n lv_iscsi vg_iscsi
  Logical volume "lv_iscsi" created
$ sudo lvs
  LV       VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_iscsi vg_iscsi -wi-ao---- 680.00g
$ sudo mkfs.gfs2 -j 2 -p lock_dlm -t gfs_cluster:gfs2 /dev/vg_iscsi/lv_iscsi
~~~

对于mkfs.gfs2命令来说，我们所使用的参数功能如下：

* -p：用来指定gfs的锁机制，一般情况下会选择lock_dlm，如果要选择类型，可以参考：online.
* -j：指定journal个数(可加入节点数)，一般情况下应留有冗余，否则后期还得再调整；
* 查看journals：

~~~ bash
$ sudo gfs2_tool journals /data3
~~~

* 增加journals：

~~~ bash
$ sudo gfs2_jadd -j1 /data3
~~~

* -t：格式为ClusterName:FS_Path_Na
* ClusterName：应与前面cluster.conf中指定的集群名称相同；
* FS_Path_Name：这个块设备mount的路径；
* 最后一个参数是指定逻辑卷的详细路径；

### 1.3 启动GFS2服务

~~~ bash
$ sudo mkdir /gfs2
$ sudo echo "/dev/vg_iscsi/lv_iscsi /gfs2 gfs2 rw,relatime 0 0" >> /etc/fstab
$ sudo service gfs2 start
~~~

### 1.4 配置开机自启动

~~~ bash
$ sudo echo 'iscsiadm -m node -T <iSCSI的IQN> -p 192.168.0.110 --login' >> /etc/rc.local
$ sudo chkconfig --add cman
$ sudo chkconfig --add clvmd
$ sudo chkconfig --add gfs2
$ sudo chkconfig --add rgmanager
$ sudo chkconfig --level 345 cman on
$ sudo chkconfig --level 345 clvmd on
$ sudo chkconfig --level 345 gfs2 on
$ sudo chkconfig --level 345 rgmanager on
~~~

## 2. 后续工作

* 验证本部署方案可能存在的隐患，例如单点故障、脑裂
* 验证iSCSI多路径的配置，增强整体的稳定性
* 验证防火墙配置
* 验证lvm的配置影响及更深入的配置
* 验证集群配置
* 验证mkfs.gfs2的相关选项
