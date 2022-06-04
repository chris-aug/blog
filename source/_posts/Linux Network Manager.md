---
title: Linux Network Manager
tags: Network
categories: Linux
cover: /img/cover/e08c6853.webp
abbrlink: e08c6853
date: 2021-08-29 23:02:08
description:  Linux 网络管理命令行工具的基本使用
sticky: 
comments:
katex: 
aplayer: 
---
![封面图](/img/cover/e08c6853.webp)
## 网络主机名设置
显示主机名
```
hostname
hostnamectl status
```
设置主机名
```
hostnamectl set-hostname Hostname
```

## 网络管理命令行工具 - nmcli
nmcli 的全称是，NetworkManager Command Line Tool - 网络管理命令行工具。nmcli 是一个非常丰富和灵活的命令行工具，NetworkManager 是管理和监控网络设置的守护进程，设备既就是网络接口，连接是对网络接口的配置，一个网络接口可以有多个连接配置，不过只能由一个连接生效。同时 nmcli 对网络的配置是可以直接写入配置文件的。

### Device 设备管理
```
nmcli device
```
### Connection 连接管理
```
nmcli connection
```
### 图形工具
```
nm-connection-editor
```
### 字符工具
```
	nmtui
	nmtui-connect
	nmtui-edit
	nmtui-hostname
```
### 创建连接（default;DHCP）
```shell
$ nmcli connection add con-name default type Ethernet ifname eth0
```
### 删除连接
```shell
$ nmcli connection defult delete
```
### 创建连接（指定参数：固定 IP 地址不自动连接）
```shell
$ nmcli connection add con-name TEST2 ipv4.method manual finame ens33 autoconnect no type Ethernet ipv4.adderess 172.16.16.100/24 gw4 172.16.16.1
```
### 修改网卡配置
示例：修改网卡自动启动
```shell
$ nmcli connection modify ens33 connection.autoconnect yes
```
**参数说明**

| 参数                              | 对应值                          |
|:--------------------------------:|:------------------------------:|
| ipv4.method manual               | BOOTPROTO=none                 |
| ipv4.method auto　               | BOOTPROTO=dhcp                  |
| ipv4.addresses "192.0.2.1/24     | IPADDR=192.0.2.1  PREFIX=24     |
| gw4 192.0.2.254"                 | GATEWAY=192.0.2.254             |
| ipv4.dns 8.8.8.8             　  |　DNS0=8.8.8.8                    |
| ipv4.dns-search example.com 　   |　DOMAIN=example.com              |
| ipv4.ignore-auto-dns true　　    |　PEERDNS=no                      |
| connection.autoconnect yes 　    |　ONBOOT=yes                      |
| connection.id eth0 　　　　　     |  NAME=eth0                       |
| connection.interface-name eth0　 |　DEVICE=eth0                     |
| 802-3-ethernet.mac-address . . . |　HWADDR= . . .                   | 

## Linux Network Proxy
部分情况下使用 Konsole 或者其 Terminal 会对部分服务使用本地代理，但在这篇文章中仅叙述常见的且使用需求较高的终端和 Git 代理的设置方法。

### 终端代理
使用如下命令
```shell
export http_proxy=http://ProxyAddress:port
export http_proxy=socks5://ProxyAddress:port
```
举例
```shell
export http_proxy=http://127.0.0.1:9999
export https_proxy=socks5://127.0.0.1:6666
```
### Git 代理
Git 代理的使用中和其他配置一样按需求使用 `--global` 参数，同时地址部分也可以使用本地 HTTP 或本地 Socks5. 在下面仅举例其中一种。
```shell
git config --global http.proxy 'socks5://127.0.0.1:1080' 
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

