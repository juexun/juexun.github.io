---
layout: post
title: Python 开发环境
modified:
categories: coding
excerpt: "Python编程相关的软件"
tags: [Python]
image:
  feature:
date: 2016-02-19T22:52:09+08:00
---

{% include _toc.html %}

在使用任何一门计算机语言进行开发之前，我们都需要构建相应的开发环境。例如Windows上我们常用的IDE就是VS，里面包含了代码编辑器，编译器和运行库等组件。

使用Python来开发软件前，我们也必须要安装开发环境：Python解析器；配合上最简陋的文本编辑器，就可以编写和运行程序了。

## 0. Python脚本解释器

目前，Python已经有多种不同版本的解释器程序；在Linux平台下，已经默认支持了Python。例如CentOS 6.x，已经默认安装了Python 2.6解释器。如果没有特殊的要求，可以直接用vim就编码了。

## 1. 包管理工具PIP

之前我们提到过，Python一个最大的特点就是：有极其丰富的第三方库。这些第三方库我们在[PYPI](https://pypi.python.org/pypi)上搜索，并使用PIP工具进行安装。

~~~ bash
$ sudo pip install <模块名称>
~~~

## 2. 修改PYPI默认源

本文以豆瓣源为例
{: .notice}

~~~ bash
$ cat ~/.pip/pip.conf
[global]
index-url = http://pypi.douban.com/simple
trusted-host = pypi.douban.com
~~~

## 3. 修改setup.py的默认源

~~~ bash
$ cat ~/.pydistutils.cfg
[easy_install]
index_url = http://pypi.douban.com/simple
trusted-host = pypi.douban.com
~~~

## 4. 虚拟环境Virtualenv

python的另一个特性是：虚拟环境，也就是程序沙盒。这个特点其实是通过一个叫[**virtualenv**](https://pypi.python.org/pypi/virtualenv/)的第三方库实现的。
