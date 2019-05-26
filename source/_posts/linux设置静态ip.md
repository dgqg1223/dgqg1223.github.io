---
title: vue环境安装
date: 2019-03-20 11:59:05
tags: Linux
categories: 
    - Linux
---

1. 查看网卡名
```
ifconfig
```
2. 修改linux网卡配置信息 ifcfg-eth0为对应的网卡配置
```
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
系统默认设置
```
DEVICE="eth0"
BOOTPROTO="dhcp"
HWADDR="00:0C:29:F7:92:50"
IPV6INIT="yes"
NM_CONTROLLED="yes"
ONBOOT="yes"
TYPE="Ethernet"
UUID="1eebd779-f4c1-4ae8-81ed-de91fe8f934f"
```
3. 修改 ifcfg-eth0 中的信息

```
#网络类型：以太网，默认不修改
TYPE="Ethernet"
#ip状态 自动获取：dhcp  静态：static
BOOTPROTO="static"
#网卡名 默认不修改
DEVICE="eth0"
#mac地址使用系统生成的不修改
HWADDR="00:0C:29:F7:92:50"
#ipv6是否开启
IPV6INIT="yes"
#network manger参数，表示配置实时生效，修改后无需要重启网卡立即生效
NM_CONTROLLED="yes"
#UUID 使用系统生成的 不修改
UUID="1eebd779-f4c1-4ae8-81ed-de91fe8f934f"
#系统启动时候是否激活网卡 设置为yes
ONBOOT="yes"
#静态ip地址
IPADDR="192.168.2.1"
#子掩码
NETMASK="255.255.255.0"
#网关
GATEWAY="192.168.2.253"
#dns
DNS1="192.168.2.253"
```


