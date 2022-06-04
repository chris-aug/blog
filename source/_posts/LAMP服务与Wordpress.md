---
title: LAMP 服务与 Wordpress
tags: Wordpress
categories: Tips
cover: /img/cover/a1901184.webp
abbrlink: a1901184
date: 2021-08-27 19:48:42
description:  LAMP 的搭建与 Wordpress 基础功能部署
sticky: 
comments:
katex: 
aplayer: 
---

![封面图](/img/cover/a1901184.webp)

## 当前文章实现环境
- CentOS 8 
- Apache `Httpd 下文使用 Apache 名称`
- MariaDB `与 MySQL 完全兼容 下文只使用 MariaDB 名称`
- Wordpress 5.8
- **如果你使用其他环境请自行根据实际需求调整**

## LAMP
### Apache & MariaDB
检查系统是否有安装Apache MariaDB
```shell
rpm -qa | grep "mariadb*"
```
卸载系统版本重装
```shell
yum -y remove "mariadb*"
yum -y install mariadb-server
```

启动软件与设置开机自启
`systemctl start mariadb`
`systemctl enable mariadb`

### PHP
检查安装并移除已安装
```shell
rpm -qa | grep "php*"
yum remove "php*"
```
#### 配置源
```shell
vim /etc/yum.repos.d/CentOS-AppStream.repo
[AppStream]
name=CentOS-$releasever - AppStream
baseurl=http://mirrors.aliyun.com/centos/$releasever/AppStream/$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

vim /etc/yum.repos.d/CentOS-Base.repo
[BaseOS]
name=CentOS-$releasever - Base
baseurl=http://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial

vim /etc/yum.repos.d/CentOS-Epel.repo
[epel]
name=CentOS-$releasever - Epel
baseurl=http://mirrors.aliyun.com/epel/8/Everything/$basearch
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
```
#### 清理并重新建立缓存
```shell
yum clean all
yum makecache
```
#### 安装 epel & remi 存储库并检查
```shell
epel
dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
rpm -qa | grep epel

remi
dnf -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
rpm -qa | grep remi
```
#### 获取 PHP 模块流列表
```shell
dnf module list php
```
重置PHP模块与配置对应版本的流模块
```shell
dnf module reset php
dnf module enable php:remi-7.4
```

#### 安装 PHP
Wordpress 需要的模组，[官方文档](https://make.wordpress.org/hosting/handbook/server-environment/#php-extensions)。
>PHP Extensions
WordPress core makes use of PHP extensions. If the preferred extension is missing WordPress will either have to do more work to do the task the module helps with or, in the worst case, will remove functionality. Therefore the PHP extensions listed below are recommended.
curl – Performs remote request operations.
dom – Used to validate Text Widget content and to automatically configure IIS7+.
exif – Works with metadata stored in images.
fileinfo – Used to detect mimetype of file uploads.
hash – Used for hashing, including passwords and update packages.
imagick – Provides better image quality for media uploads. See WP_Image_Editor for details. Smarter image resizing (for smaller images) and PDF thumbnail support, when Ghost Script is also available.
json – Used for communications with other servers.
mbstring – Used to properly handle UTF8 text.
mysqli – Connects to MySQL for database interactions.
openssl – Permits SSL-based connections to other hosts.
pcre – Increases performance of pattern matching in code searches.
sodium – Validates Signatures and provides securely random bytes.
xml – Used for XML parsing, such as from a third-party site.
zip – Used for decompressing Plugins, Themes, and WordPress update packages.

安装命令为
`yum install php php-mbstring php-pecl-zip php-xml php-mysqlnd php-pecl-imagick-im6-devel php-json php-pdo php-opcache php-fpm php-bcmath php-pecl-imagick-im6 php-common php-sodium php-devel`

>亦可使用 dnf install 安装所需 Package 是一样的
>检查安装与查看版本命令 Packagename -v

## 数据库 MariaDB

### 安装数据库
```shell
yum install -y mariadb-server
```

### 配置数据库安全设置
```shell
mysql_secure_installation
```

### 配置 Wordpress 数据库
```sql
mysql -uroot -p
create database dbname;
create user 'user' @ 'localhost' identified by 'password';
grant all privileges on dbname.* to 'user' @ 'localhost' identified by 'password';
flush privileges;
```
## 配置 Apache & Wordpress
### 获取 Wordpress 源码
```shell
cd /var/www/html
wget https://cn.wordpress.org/latest-zh_CN.tar.gz
tar zxvf latest-zh_CN.tar.gz
cp /wordpress/* /var/www/html #非必需
rm latest-zh_CN.tar.gz        #非必需
rm -r wordpress               #非必需
```
### 修改网站目录权限
```shell
chown -R www-data /var/www/html/ #Ubuntu
chown -R apache /var/www/html/   #CentOS
```

### 设置 Wordpress
现在你可以使用浏览器访问 Wordpress 设置页面了，网址如下：
```
http://(e.g[1.1.1.1]) or http://URL
```
填入数据库信息以及管理员等账户信息
![首次登陆后台设置页面](https://main.qcloudimg.com/raw/c79c35b3d75f763561d7024f46983611.png)

### Apache 配置文件
#### HTTP 虚拟主机
```shell
<VirtualHost *:80>
  ServerName www.example.com    #域名
  ServerAlias example.com       #其他域 e.g自域 www
  DocumentRoot "/var/www/html"  #网站目录
 <Directory /var/www/html/>     #网站目录
  Options -Indexes +FollowSymLinks #防止目录列表和告诉Web服务器遵循符号链接
  AllowOverride All	 #指定.htaccess文件中声明的指令可以覆盖配置指令
 </Directory>

  ErrorLog /var/log/httpd/example.com-error.log  #日志文件
  CustomLog /var/log/httpd/exampale-access.log combined #日志文件
</VirtualHost>
```
[更多请参考 Apache Documentation Version2.4 Vhosts 官方文档](https://httpd.apache.org/docs/2.4/vhosts/examples.html)

### HTTPS
1、准备证书文件
>免费证书以腾讯云为例
>解压证书文件后文件夹名称：Apache
文件夹内容：
1_root_bundle.crt 证书文件
2_cloud.tencent.com.crt 证书文件
3_cloud.tencent.com.key 私钥文件

使用远程文件管理软件例如 WinSCP 将文件上传到 Apache 服务器 /etc/httpd/ssl
若文件夹不存在使用 `mkdir /etc/httpd/ssl` 创建

>首次安装的 Apache 服务器，conf.d、conf、conf.modules.d 等目录默认在 /etc/httpd 目录下

2、在 /etc/httpd/conf 目录下的 httpd.conf 配置文件找到 Include conf.modules.d/*.conf（用于加载配置 SSL 的配置目录）配置语句，并确认该配置语句未被注释。若已注释，请去掉首行的注释符号（#），保存配置文件。

3、在 /etc/httpd/conf.modules.d 目录下的 00-ssl.conf 配置文件找到 LoadModule ssl_module modules/mod_ssl.so（用于加载 SSL 模块）配置语句，并确认该配置语句未被注释，若已注释，请去掉首行的注释符号（#），保存配置文件。
安装mod_ssl `yum install mod_ssl`
>由于操作系统的版本不同，目录结构也不同，请根据实际操作系统版本进行查找。
若以上配置文件中均未找到 LoadModule ssl_module modules/mod_ssl.so 和 Include conf.modules.d/*.conf 配置语句，请确认是否已经安装 mod_ssl.so 模块。若未安装 mod_ssl.so 模块，您可通过执行yum install mod_ssl 命令进行安装。

4、编辑HTTPS配置文件
编辑 /etc/httpd/conf.d 目录下的 ssl.conf 配置文件。修改如下内容：
```shell
<VirtualHost 0.0.0.0:443>
 DocumentRoot "/var/www/html" 
 #填写证书名称
 ServerName cloud.tencent.com 
 #启用 SSL 功能
 SSLEngine on 
 #证书文件的路径
 SSLCertificateFile /etc/httpd/ssl/2_cloud.tencent.com.crt 
 #私钥文件的路径
 SSLCertificateKeyFile /etc/httpd/ssl/3_cloud.tencent.com.key 
 #证书链文件的路径
 SSLCertificateChainFile /etc/httpd/ssl/1_root_bundle.crt 
</VirtualHost>
```
### HTTP 自动跳转 HTTPS 的安全配置（可选）
1、编辑 /etc/httpd/conf 目录下的 httpd.conf 配置文件
确认该配置文件是否存在LoadModule rewrite_module modules/mod_rewrite.so并取消注释
2、在HTTP虚拟主机配置文件中添加重写参数

```shell
ServerName www.example.com
    ServerAlias example.com    
    RewriteEngine On
    RewriteCond %{HTTPS} !=on
    RewriteRule ^(.*) https://%{SERVER_NAME}$1 [L,R]
```
完整参考
```shell
<VirtualHost *:80>
  ServerName www.example.com    #域名
  ServerAlias example.com       #其他域 e.g自域 www
  DocumentRoot "/var/www/html"  #网站目录
 <Directory /var/www/html/>     #网站目录
  Options -Indexes +FollowSymLinks #防止目录列表和告诉Web服务器遵循符号链接
  AllowOverride All			#指定.htaccess文件中声明的指令可以覆盖配置指令
 </Directory>

  ErrorLog /var/log/httpd/example.com-error.log  #日志文件
  CustomLog /var/log/httpd/exampale-access.log combined #日志文件

 RewriteEngine On
 RewriteCond %{HTTPS} !=on
 RewriteRule ^(.*) https://%{SERVER_NAME}$1 [L,R]

</VirtualHost>
```
>解释：
RewriteEngine On 是开启 rewrite 功能
RewriteCond %{HTTPS} !=on  为不是https的时候执行下面的规则
^(.*) https://%{SERVER_NAME}$1 [L,R] 中 ^ 匹配行的开始 
$1引用 RewriteRule 中的第一个正则(.*)代表的字符， %{SERVER_NAME} 就是监听的网站域名，[L]：结尾标识。停止重写操作，并不再应用其他重写规则。防止本条规则被后续规则影响
R 强制外部重定向

3、如在步骤1中未找到相关参数
直接 /etc/httpd/conf.modules.d 中新建一个  *.conf  文件，e.g 00-rewrite.conf 添加一下内容
`LoadModule rewrite_module modules/mod_rewrite.so`