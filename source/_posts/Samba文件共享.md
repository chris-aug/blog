---
title: Samba 文件共享
tags: Samba
categories: Linux
cover: /img/cover/111902ff.webp
abbrlink: 111902ff
date: 2021-10-27 14:28:48
description: Samba 网络文件共享基础设置
sticky:
comments:
katex:
aplayer:
---

![封面图](/img/cover/111902ff.webp)

## 前言

基于我个人是 [使用 Linux 作为主系统](https://blog.blankcoder.com/posts/81acf3e4.html) ，同时在上篇文章 [Manjaro & Arch 配置 KVM](https://blog.blankcoder.com/posts/8fa8ffbb.html) 中，将虚拟机从 VMware 替换为了开源 KVM 后，与 Windows 虚拟机之间的文件共享暂时还没有想到好的解决办法，经过考虑决定以最小化配置 1C512M 的配置搭建一个在宿主机和多台虚拟机均可直接访问的文件共享服务。

## 安装 Samba
以 CentOS 为例，使用 `yum` 或者 `dnf` 安装即可，依照所使用发行版的软件安装方式安装即可。

## 查询软件安装
```
#以 CentOS 为例
rpm -qa | grep "samba"
#以 Arch 系为例
pacman -Qs samba
```
## 共享形式
### 匿名访问配置
>Global 为全局配置，Disk 为共享配置。
>匿名访问是不设限制的文件访问，任何能访问到共享主机的设备都可以读取或写入。
```sh
[global]
        workgroup = SAMBA	#工作组名称        
        security = user
        map to guest = Bad User #此处两行为安全等级        
        passdb backend = tdbsam #此部分为用户后台模式

        #此部分为打印共享配置，仅文件共享可以注释禁用
        printing = cups			#打印类型
        printcap name = cups 	#打印机的配置文件
        load printers = yes		#是否开启打印机共享
        cups options = raw

		#此部分为文件与目录创建权限 （根据安全需求设置，我这里为公开型）
        create mask = 0777 		#创建文件权限为0777
        directory mask  = 0777	#创建目录权限为0777

[Disk]
        comment = Samba 	#目录说明
        path = /disk		#共享路径
        writable = yes		#是否可写入		
        guest ok = yes		#是否公开

```
### 指定用户访问
>指定用户通过帐号密码访问文件夹。

用户配置部分：
```
$ groupadd samba	#创建用户组
$ useradd -g samba sambausr #创建用户
$ smbpasswd -a sambausr	#创建smaba访问密码
```

共享文件夹操作部分：

```
以home下对应用户目录为例：
$ chcon -t samba_share_t /home/sambausr #SELiux安全上下文
$ chown -R sambausr:samba /home/sambausr #修改文件所有者和文件关联组
```

Samba 共享配置部分：
```
[global]
        workgroup = SAMBA	#工作组名称        
        security = user
        hosts allow = 10.10.10.1 #允许访问主机        
        passdb backend = tdbsam #此部分为用户后台模式

[Disk]
        comment = Samba 	#目录说明
        path = /home/sambausr		#共享路径
        writable = yes		#是否可写入
        guest ok = no		#是否公开
```

### 用户组共享方式
>懒，有人留言再来写🤣

## 重启服务启用文件共享
```
systemctl restart smb
```

## 访问与挂载
### Windows
使用运行（Win+R），输入`\\ip` ，根据弹窗进行下一步操作即可。 
### Linux
匿名访问配置访问：
```
$  mount -t cifs //server/Folder /mnt/
```
用户访问配置访问：
```
$ mount -t cifs //server/Folder /mnt/ -o username=your-username
```
**提示：**提示输入密码时，匿名访问配置直接 Enter，用户访问配置输入用户 smb 密码。

## 无法访问排查
（可能更新）
### Firewalld 防火墙
>此处建议不关闭防火墙，而允许 Samba 穿越防火墙，以便外部用户可以访问 Samba 共享。
```
$ firewalld-cmd --permanent --add-service=samba
$ firewaldd-cmd --reload
```
### SELinux 安全上下文
>在 CentOS 上 SELinux 是默认安装开启的，通常情况下都会导致 Samba, FTP 等应用无法访问相应目录，此处就 Samba 服务的解决方案如下：
```
#放行 Samba 用户 Home 目录权限：
$ /usr/sbin/setsebool -P samba_enable_home_dirs=1
#指定路径放行
$ chcon -t samba_share_t path
#查看目录是否开启权限
$ ls -ldZ path
```
