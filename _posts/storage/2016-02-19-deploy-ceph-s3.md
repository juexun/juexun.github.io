---
layout: post
title: 部署 Ceph S3 网关
modified:
categories: storage
excerpt: "基于Ceph集群部署S3网关流程"
tags: [Ceph]
comments: true
image:
  feature:
date: 2016-02-19T12:24:06+08:00
---

{% include _toc.html %}

## 0. 前言

在Cep官方文档中[安装Ceph对象网关部分](http://docs.ceph.com/docs/v0.80.5/install/install-ceph-gateway/)，介绍的安装流程是先要安装httpd和mod_fastcgi，然后再安装ceph和ceph-radosgw。其实，这个先后顺序没有必然联系。由于我已经安装了Ceph和radosgw，这里就不再重新安装了。

本文只介绍基于CentOS 6.x的单节点对象网关安装流程，关于联邦架构的对象网关模式，会在后续的文章中继续跟进。

----

## 1. 安装Apache和FastCGI

根据官方文档提供的资料，Ceph对象网关需要支持“100-continue”的Apache和FastCGI。标准版本并没有支持该协议；Ceph社区发布了支持该选项的版本。

> 100-continue用于客户端在发送POST数据给服务器前，征询服务器情况，看服务器是否处理POST的数据，如果不处理，客户端则不上传POST数据，如果处理，则POST上传数据。
> 
> 在现实应用中，通过在POST大数据时，才会使用100-continue协议。

### 1.0 配置YUM源

* 考虑到网络及可能需要反复安装的问题，结合上次安装的经验，我还是把软件包下载到本地服务器后制作本地源
  
  * Apache下载链接：http://gitbuilder.ceph.com/httpd-rpm-centos6-x86_64/ref/master/
  * FastCGI下载链接：http://gitbuilder.ceph.com/mod_fastcgi-rpm-centos6-x86_64-basic/ref/master/
  
* 定义YUM源
  
  * Apache源
    
~~~ bash
$ sudo cat /etc/yum.repos.d/apache2-ceph.repo
[apache2-ceph-noarch]
name=Apache noarch packages for Ceph
baseurl=<本地源URL>
enabled=1
priority=2
gpgcheck=0
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc
~~~
    
  * FastCGI源
    
~~~ bash
$ sudo cat /etc/yum.repos.d/fastcgi-ceph.repo
[fastcgi-ceph-basearch]
name=FastCGI basearch packages for Ceph
baseurl=<本地源URL>
enabled=1
priority=2
gpgcheck=0
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc

[fastcgi-ceph-noarch]
name=FastCGI noarch packages for Ceph
baseurl=<本地源URL>
enabled=1
priority=2
gpgcheck=0
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/autobuild.asc
~~~
  
### 1.1 安装
  
~~~ bash
$ sudo yum install httpd mod_fastcgi
~~~

## 2. 配置Apache和FastCGI

* 配置Apache
  
  * 编辑Apache配置文件
  * 配置ServerName
  * 加载Rewrite模块
  * 配置FastCGI
    
~~~ bash
$ sudo cat /etc/httpd/conf/httpd.conf

ServerName {fgdn}

LoadModule rewrite_module modules/mod_rewrite.so
LoadModule fastcgi_module modules/mod_fastcgi.so
~~~

**注:** fgdn为主机的FQDN，通过以下命令获取: **hostname -f**
{: .notice}
  
* 重启Apache服务
  
~~~ bash
$ sudo /etc/init.d/httpd restart
~~~

## 3. 配置SSL

可跳过

## 4. 配置DNS

可跳过

## 5. 安装Ceph对象网关

* 安装对象网关服务
  
~~~ bash
$ sudo yum install ceph-radosgw ceph
~~~
  
* 安装对象网关代理
  
~~~ bash
$ sudo yum install radosgw-agent
~~~

## 6. 配置Ceph对象网关

原文链接：http://docs.ceph.com/docs/master/radosgw/config/

### 6.0 生成keyring和key

* 生成keyring
  
~~~ bash
$ sudo ceph-authtool --create-keyring /etc/ceph/keyring.radosgw.gateway
$ sudo chmod +r /etc/ceph/keyring.radosgw.gateway
~~~


* 生成key
  
~~~ bash
$ sudo ceph-authtool /etc/ceph/keyring.radosgw.gateway -n client.radosgw.gateway --gen-key
$ sudo ceph-authtool -n client.radosgw.gateway --cap osd 'allow rwx' --cap mon 'allow rw' /etc/ceph/keyring.radosgw.gateway
~~~
  
* 添加keyring项
  
~~~ bash
$ sudo ceph -k /etc/ceph/ceph.client.admin.keyring auth add client.radosgw.gateway -i /etc/ceph/keyring.radosgw.gateway
~~~


* 如果对象网关和ceph服务器不是同一台机器，需要拷贝keyring文件到对象网关服务器上

## 7. 创建默认池

创建以下默认存储池

- .rgw.root
- .rgw.control
- .rgw.gc
- .rgw.buckets
- .rgw.buckets.index
- .rgw.buckets.extra
- .log
- .intent-log
- .usage
- .users
- .users.email
- .users.swift
- .users.uid

### 7.0 增加对象网关配置到Ceph配置文件

* 在ceph.conf中加入如下内容
  
~~~ text
[client.radosgw.gateway]
      host = {Ceph对象网关主机名，注：不是FQDN}
      keyring = /etc/ceph/keyring.radosgw.gateway
      rgw socket path = /tmp/radosgw.sock
      log file = /var/log/ceph/radosgw.log
~~~
  
* 推送配置到集群里的其他服务器
  
~~~ bash
$ ceph-deploy config push {host-name [host-name]...}
~~~

### 7.1 创建数据目录

命令格式如下：

~~~ bash
$ sudo mkdir -p /var/lib/ceph/radosgw/{$cluster}-{$id}
~~~

根据以上配置文件定义，实际执行的命令为：

~~~ bash
$ sudo mkdir -p /var/lib/ceph/radosgw/ceph-radosgw.gateway
~~~

设置日志文件权限

~~~ bash
$ sudo /etc/init.d/ceph-radosgw start
$ sudo chown apache:apache /var/log/radosgw/client.radosgw.gateway.log
(这个文件在第一次radosgw启动过之后才会创建)
$ sudo /etc/init.d/ceph-radosgw restart
~~~

### 7.2 创建网关配置

* 新增/etc/httpd/conf.d/rgw.conf文件，内容如下：
  
~~~ 
FastCgiExternalServer /var/www/s3gw.fcgi -socket /tmp/radosgw.sock

<VirtualHost *:80>
ServerName rgw.example1.com
ServerAlias rgw
ServerAdmin webmaster@example1.com
DocumentRoot /var/www

RewriteEngine On
RewriteRule ^/([a-zA-Z0-9-_.]*)([/]?.*) /s3gw.fcgi?page=$1&params=$2&%{QUERY_STRING} [E=HTTP_AUTHORIZATION:%{HTTP:Authorization
},L]

<IfModule mod_fastcgi.c>
  <Directory /var/www>
    Options +ExecCGI
    AllowOverride All
    SetHandler fastcgi-script
    Order allow,deny
    Allow from all
    AuthBasicAuthoritative Off
  </Directory>
</IfModule>

AllowEncodedSlashes On
ServerSignature Off
</VirtualHost>
~~~


* 在CentOS及同类平台下，关闭FastCgiWrapper选项
  
~~~ bash
$ sudo vim /etc/httpd/conf.d/fastcgi.conf
~~~
  
将FastCgiWrapper设置为Off

## 8. 重启服务

~~~ bash
$ sudo service ceph restart
$ sudo service ceph-radosgw restart
$ sudo service httpd restart
~~~

## 9. 使用网关

* 创建用户用于S3访问
  
~~~ bash
$ sudo radosgw-admin user create --uid="testuser" --display-name="First User"
{"user_id": "testuser",
"display_name": "First User",
"email": "",
"suspended": 0,
"max_buckets": 1000,
"auid": 0,
"subusers": [],
"keys": [
{ "user": "testuser",
"access_key": "I0PJDPCIYZ665MW88W9R",
"secret_key": "dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA"}],
"swift_keys": [],
"caps": [],
"op_mask": "read, write, delete",
"default_placement": "",
"placement_tags": [],
"bucket_quota": { "enabled": false,
"max_size_kb": -1,
"max_objects": -1},
"user_quota": { "enabled": false,
"max_size_kb": -1,
"max_objects": -1},
"temp_url_keys": []}
~~~


* 创建SWIFT用户
  
  * 创建用户
    
~~~ bash
$ sudo radosgw-admin subuser create --uid=testuser --subuser=testuser:swift --access=full
{ "user_id": "testuser",
"display_name": "First User",
"email": "",
"suspended": 0,
"max_buckets": 1000,
"auid": 0,
"subusers": [
{ "id": "testuser:swift",
"permissions": "full-control"}],
"keys": [
{ "user": "testuser:swift",
"access_key": "3Y1LNW4Q6X0Y53A52DET",
"secret_key": ""},
{ "user": "testuser",
"access_key": "I0PJDPCIYZ665MW88W9R",
"secret_key": "dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA"}],
"swift_keys": [],
"caps": [],
"op_mask": "read, write, delete",
"default_placement": "",
"placement_tags": [],
"bucket_quota": { "enabled": false,
"max_size_kb": -1,
"max_objects": -1},
"user_quota": { "enabled": false,
"max_size_kb": -1,
"max_objects": -1},
"temp_url_keys": []}
~~~
    
  * 创建secret key
    
~~~ bash
$ sudo radosgw-admin key create --subuser=testuser:swift --key-type=swift --gen-secret
{ "user_id": "testuser",
"display_name": "First User",
"email": "",
"suspended": 0,
"max_buckets": 1000,
"auid": 0,
"subusers": [
{ "id": "testuser:swift",
"permissions": "full-control"}],
"keys": [
{ "user": "testuser:swift",
"access_key": "3Y1LNW4Q6X0Y53A52DET",
"secret_key": ""},
{ "user": "testuser",
"access_key": "I0PJDPCIYZ665MW88W9R",
"secret_key": "dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA"}],
"swift_keys": [
{ "user": "testuser:swift",
"secret_key": "244+fz2gSqoHwR3lYtSbIyomyPHf3i7rgSJrF\/IA"}],
"caps": [],
"op_mask": "read, write, delete",
"default_placement": "",
"placement_tags": [],
"bucket_quota": { "enabled": false,
"max_size_kb": -1,
"max_objects": -1},
"user_quota": { "enabled": false,
"max_size_kb": -1,
"max_objects": -1},
"temp_url_keys": []}
~~~


* 测试S3
  
  * 安装python-boto
    
  * 编辑s3test.py测试代码，内容如下
    
~~~ python
import boto
import boto.s3.connection
access_key = 'I0PJDPCIYZ665MW88W9R'
secret_key = 'dxaXZ8U90SXydYzyS5ivamEP20hkLSUViiaR+ZDA'
conn = boto.connect_s3(
aws_access_key_id = access_key,
aws_secret_access_key = secret_key,
host = '{对象网关主机名}',
is_secure=False,
calling_format = boto.s3.connection.OrdinaryCallingFormat(),
)
bucket = conn.create_bucket('my-new-bucket')
for bucket in conn.get_all_buckets():
        print "{name}\t{created}".format(
                name = bucket.name,
                created = bucket.creation_date,
)
~~~
    
  * 执行测试脚本
    
~~~ bash
$ python s3test.py
my-new-bucket 2015-02-16T17:09:10.000Z
~~~
    
    ​

## 10. 参考链接

* [官方文档](http://docs.ceph.com/docs/master/radosgw/config/)
  
* [ceph 对象存储网关rados gateway和S3接口测试详细安装配置文档](http://www.ithao123.cn/content-790311.html)