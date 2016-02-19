---
layout: post
title: Virtualenv
modified:
categories: coding
excerpt: "Virtualenv是一个类似沙盒的Python库，用于配置独立的Python运行环境。"
tags: [Python]
image:
  feature:
date: 2016-02-19T22:57:17+08:00
---

{% include _toc.html %}

Virtualenv用于创建独立的Python环境，多个Python相互独立，互不影响，它能够：

* 在没有root权限的情况下安装新套件
* 不同虚拟环境可以使用不同的套件版本
* 套件升级不影响其它虚拟环境

## 0. 安装Virtualenv库

CentOS的EPEL仓库中已经包含了Virtualenv的RPM包，我们可以通过YUM工具进行安装：

~~~ bash
$ sudo yum install python-virtualenv
~~~

但是，这种方式安装的软件版本跟官方PYPI源相比，一般版本会低很多。所以，我们也可以通过PIP工具直接从PYPI源安装。

~~~ bash
$ sudo pip install virtualenv
~~~

## 1. 创建虚拟环境

Virtualenv库安装完成后，我们就可以创建虚拟环境了。

*注：使用Virtualenv创建虚拟环境是不需要root权限*

~~~ bash
$ virtualenv <虚拟环境路径>
~~~

### 1.0 命令参数

| 选项                 | 用途         | 备注                                       |
| :------------------ | :----------: | ----------------------------------------: |
| --no-site-packages | 不依赖系统中已安装库 | 所有依赖的库都必须安装，这种方式生成的虚拟环境只包含Python标准库，所有非标准库都必须在虚拟环境中重新安装 |

## 2. 启动虚拟环境

虚拟环境创建成功后，我们就可以启动并在虚拟环境中运行Python脚本了。

~~~ bash
$ source <虚拟环境路径>/bin/active
[虚拟环境名称]$
~~~

启动虚拟环境后，所有与Python相关的操作都是基于这个虚拟环境来完成的。

例如，在虚拟环境启动后，使用PIP安装第三方库，只会对当前虚拟环境产生影响，不会影响到其它虚拟环境；运行Python脚本只能使用当前虚拟环境中已安装的库等。

## 3. 退出当前虚拟环境

~~~ bash
$ deactive
~~~
