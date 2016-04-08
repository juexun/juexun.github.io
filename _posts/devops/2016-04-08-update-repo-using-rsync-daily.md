---
layout: post
title: 同步CentOS YUM源
modified:
categories: devops
excerpt: "使用RSYNC工具定时同步CentOS YUM源"
tags: []
image:
  feature:
date: 2016-04-08T19:13:35+08:00
---

## 编写Shell脚本

## 将脚本存储定时任务目录

## 重启定时任务crond服务

~~~
$ sudo systemctl restart crond
~~~

## 脚本内容

~~~ bash
#!/bin/bash

# base value
# 要同步的源
MIRROR_SITE="rsync://mirrors.yun-idc.com/"
# 本地存放目录
LOCAL_PATH="/home/backup/repo/"
# 需要同步的软件
LOCAL_REPO='centos epel'
# 需要同步的版本
LOCAL_VER="7.2.1511 7"
# 记录本脚本进程号
LOCK_FILE="/var/log/rsyncrepo.pid"
# 同步日志文件
LogFile=/var/log/rsyncrepo/`date +"%Y-%m-%d"`.log
# 如用系统默认rsync工具为空即可。
# 如用自己安装的rsync工具直接填写完整路径
RSYNC_PATH=""
 
# check update yum server  pid
MY_PID=$$
if [ -f $LOCK_FILE ]; then
    get_pid=`/bin/cat $LOCK_FILE`
    get_system_pid=`/bin/ps -ef|grep -v grep|grep $get_pid|wc -l`
    if [ $get_system_pid -eq 0] ; then
        echo $MY_PID>$LOCK_FILE
    else
        echo "Have update yum server now!"
        exit 1
    fi
else
    echo $MY_PID>$LOCK_FILE
fi
 
# check rsync tool
if [ -z $RSYNC_PATH ]; then
    RSYNC_PATH=`/usr/bin/whereis rsync|awk ' ''{print $2}'`
    if [ -z $RSYNC_PATH ]; then
        echo 'Not find rsync tool.'
        echo 'use comm: yum install -y rsync'
    fi
fi
 
# sync yum source
echo "rsync start at $(date +"%Y-%m-%d %H:%M:%S")" >$LogFile
echo "--------------------------------------------------" >>$LogFile
for REPO in $LOCAL_REPO;
do
echo "update $REPO ..."
for VER in $LOCAL_VER;
do
    # Check whether there are local directory
    if [ ! -d "$LOCAL_PATH$VER" ] ; then
        echo "Create dir $LOCAL_PATH$REPO/$VER"
        `/bin/mkdir -p $LOCAL_PATH$REPO/$VER`
    fi
    # sync yum source
     echo "Start sync $LOCAL_PATH$REPO/$VER"  >>$LogFile
     echo "--------------------------------------------------" >>$LogFile 
    $RSYNC_PATH -avrtH --delete --delete-excluded --exclude=atomic --exclude=cr --exclude=fasttrack --exclude=extras --exclude=centosplus --exclude=cloud --exclude=isos --exclude=SRPMS --exclude=ppc64 --exclude=ppc64le $MIRROR_SITE$REPO/$VER $LOCAL_PATH$REPO  >>$LogFile
done
echo "rsync end at $(date +"%Y-%m-%d %H:%M:%S")" >>$LogFile
echo "--------------------------------------------------" >>$LogFile 
done
 
# clean lock file
`/bin/rm -rf $LOCK_FILE`
 
echo 'sync end.'
exit 1
~~~