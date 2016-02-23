---
layout: post
title: 关闭 Windows 自动地址设置
modified:
categories: network
excerpt: "开启自动地址设置时,如果DHCP获取地址失败,系统会自动设置169.254.xx.xx."
tags: [DHCP, Windows]
comments: true
image:
  feature:
date: 2016-02-18T17:32:55+08:00
---

{% include _toc.html %}

## 现象

Windows 使用DHCP方式获取IP地址失败，系统自动设置了一个169.254.xx.xx的IP，导致无法上网。
  
## 原理
  
* APIPA
  
也被称为link-local，地址区间在169.254.0.0-169.254.255.255。
    
* 自动地址配置
  
## 原因
    
  自动专用IP地址（Automatic Private IP Addressing）是windows（包括windows98，2000以及XP）系列操作系统的一个特性，支持在没有查找到DHCP服务器的情况下，计算机自动为自己分配地址。
  
  APIPA可以作为DHCP服务器失败的一个补偿方案，十分适合应用于小型局域网。如果找不到DHCP服务器（也许服务器突然死机，也许根本就不存在服务器），那么计算机会在169.254.0.0-169.254.255.255之间自动选择IP地址。此地址段是由互联网编号分配机构（IANA）定义的，作为保留IP段专用。一旦计算机为自己分配了一个IP地址，那么就可以通过TCP/IP协议同网络中的其他计算机进行通信。
  
  另外，Linux 可通过配置 nozeroconfig配置来关闭。
  
## 解决办法
  
  * 禁用单个网卡的自动地址配置功能
    * 定位注册表项HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\\<网卡名称>
    * 创建一个REG_DWORD类型的项，名称为IPAutoconfigurationEnabled
    * 将IPAutoconfigurationEnabled赋值为0
  * 禁用所有网卡的自动地址配置功能
    * 定位注册表项HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters
    * 创建一个REG_DWORD类型的项，名称为IPAutoconfigurationEnabled
    * 将IPAutoconfigurationEnabled赋值为0 
    
    


