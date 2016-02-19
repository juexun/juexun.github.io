---
layout: post
title: 镜像源
modified:
categories: coding
excerpt: "国内各类软件镜像源，例如Docker，npm, brew等"
tags: [其它]
image:
  feature:
date: 2016-02-19T23:02:04+08:00
---

{% include _toc.html %}

## 0. 各类FTP及RSYNC镜像

* http://www.mirrorservice.org

比较齐全的镜像网站，包括sourceforge

## 1. Homebrew镜像

### 1.0 清华镜像源

http://mirrors.tuna.tsinghua.edu.cn/help/#homebrew

* 替换现有上游
  
~~~ bash
$ sudo cd /usr/local
$ sudo git remote set-url origin git://mirrors.tuna.tsinghua.edu.cn/homebrew.git
$ sudo brew update
~~~
  
* 直接使用清华版Homebrew
  
~~~ bash
$ cd ~/tmp
$ git clone git://mirrors.tuna.tsinghua.edu.cn/homebrew.git
$ sudo rm -rf /usr/local/.git
$ sudo rm -rf /usr/local/Library
$ sudo cp -R homebrew/.git /usr/local/
$ sudo cp -R homebrew/Library /usr/local/
~~~
  
* 使用homebrew-science或者homebrew-python
  
~~~ bash
$ cd /usr/local/Library/Taps/homebrew/homebrew-science
$ sudo git remote set-url origin git://mirrors.tuna.tsinghua.edu.cn/homebrew-science.git
$ cd /usr/local/Library/Taps/homebrew/homebrew-python
$ sudo git remote set-url origin git://mirrors.tuna.tsinghua.edu.cn/homebrew-python.git
$ sudo brew update
~~~

## 2. NodeJS镜像源

### 2.0 卸载NodeJS

~~~ bash
#!/bin/bash
lsbom -f -l -s -pf /var/db/receipts/org.nodejs.pkg.bom \
| while read i; do
  sudo rm /usr/local/${i}
done
sudo rm -rf /usr/local/lib/node \
     /usr/local/lib/node_modules \
     /var/db/receipts/org.nodejs.*
~~~

### 2.1 配置镜像源

* 临时指定镜像源
  
~~~ bash
$ npm install --registry <镜像源URL> <包名>
~~~
  
* 永久设置镜像源
  
~~~ bash
$ sudo npm config set registry <镜像源URL>
~~~

### 2.2 国内镜像源

* [淘宝源](https://registry.npm.taobao.org)
* [NPM中国镜像站](http://registry.cnpmjs.org)

## 3. Docker镜像源

* [DaoCloud](https://www.daocloud.io)