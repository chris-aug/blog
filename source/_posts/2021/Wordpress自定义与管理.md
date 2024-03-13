---
title: Wordpress 自定义与管理
tags: Wordpress
categories: Obsolete
abbrlink: de3c5b65
date: 2021-08-28 14:47:16
description:  Wordpress 基础自定义管理与 SMTP
sticky: 
comments:
katex: 
aplayer: 
hidden: true
---
## Wordpress 搭建
参考前文：
- [LAMP 服务与 Wordpress](https://blog.blankcoder.com/posts/a1901184.html)

## Wordpress 自定义
Wordpress 可自定义的项目非常多，即使在保证不修改 Wordpress 核心文件的前提下依旧有着非常高的自定义能力，这里仅提供三种常见的修改需求。

本文参考的 [Wordpress 官方文档](https://codex.wordpress.org/Customizing_the_Login_Form)。

其他需要修改的例如网站主页，你应该参考你的主题文档。

<!--more-->

**注意：**

- 所有修改均在主题目录下的 `functions.php` 文件
- 大多数情况下文件位置应该是在 `/var/www/html/wordpress/wp-content/themes/themesname`
### 修改 Wordpress 登录页 Logo
```php
function my_login_logo() 
{ 
	?>
	<style type="text/css">#login h1 a, .login h1 a 
	{
		#引用的本地文件目录，也可以在此插入CDN链接
		background-image: url(<?php echo get_stylesheet_directory_uri(); ?>/images/site-login-logo.png); 
		height:65px;
		width:320px;
		background-size: 320px 65px;
		background-repeat: no-repeat;
		padding-bottom: 30px;
	}
	</style><?php
}

	add_action( 'login_enqueue_scripts', 'my_login_logo' );
```
**注意：**
- 存放本地文件的目录也是在主题目录下新建 images 目录，当然这个只要你和代码对应，可以随意自定义。
### 修改 Wordpress 登录页背景
```php
function login_background_image()
{
	?>
	<style type="text/css">
	body.login
	{
		#引用的本地文件目录，也可以在此插入CDN链接
		background-image: url(<? echo get_styleensheet_directory_url(); ?>/images/background.jpg); 
		height:100%  //高度100%显示
		width:100%  //宽度100%显示
		background-size: cover; //大小尺寸自适应
		background-repeat: no-repeat;
		background-position: center; //位置居中
	}
	</style><?php
}
	add_action('login_head', 'login_background_image');
```
**注意：**

- 存放本地文件的目录也是在主题目录下新建 images 目录，当然这个只要你和代码对应，可以随意自定义。
### 修改登录页 Logo URL & Logo Title
```php
	function my_login_logo_url(){return home_url();}
	add_filter('login_headerurl', 'my_login_logo_url');
	
	function my_login_logo_url_title(){retun 'Site-name';}
	add_filter('login_headertitle', 'my_login_logo_url_title');
```
## Wordpress 配置邮件发送服务
在 Wordpress 上要解决这件是有很多中方法，插件也有无数，在这里仅提供我使用过的参考。
### SMTP 服务的作用
- 站点通知——会给你所绑定的管理邮箱发送包括但不限于自动更新、用户注册、密码验证等。
- 邮件验证——包括发送注册链接、发送修改密码链接等。

### 配置 PHP SMTP
- 在配置文件末尾加上以下内容：
**注意：**
- Username, example.com 等字段需要你根据你的情况自行修改。
- 在 Password 部分，不是每一个邮箱都允许直接使用密码在 SMTP 发送邮件或者登录第三方客户端，有要求使用专用授权码的情况。
```php
//php smtp
function site_smtp( $phpmailer ) 
{
	$phpmailer->FromName = 'Username'; // 发件人昵称
	$phpmailer->Mailer = 'smtp';
	$phpmailer->Host = 'smtp.example.com';//邮件服务器地址
	$phpmailer->SMTPAuth = true;
	$phpmailer->Port = '465';//邮件服务器端口
	$phpmailer->SMTPSecure = 'ssl';
	$phpmailer->Username = 'user@example.com';//授权用户名
	$phpmailer->Password = 'password';//授权密码
	$phpmailer->From = 'user@example.com';//设置发件人邮箱地址
	
}

//配置后用wp_mail函数发送邮件
	add_action( 'phpmailer_init', 'site_smtp', 10);
	$headers = ['Content-Type: text/html; charset=UTF-8'];
	$mailSent = wp_mail($_POST['email'], '', $message, $headers);
```

## Wordpress 快速打包跑路(迁移)
如果学生党使用学生优惠购买阿里云、腾讯云等学生专享类云服务器机型，可能会面临三年优惠结束，续费价格相比优惠时期高达数十倍。此时打包跑路便非常有用了，你可以快速将文件打包然后更换云服务器提供商或者更换云服务器地区。

>此方法打包搬家要求 WordPress 两端服务器 SQL 数据库和 PHP 环境配置完全一致。

### 数据库
#### 导出数据库
Linux 使用 `mysqldump` 命令来导出数据库，格式语法如下：
```shell
mysqldump -u root -p database_name > database_name.sql
```

> 只导出表结构需要加上 -d

e.g

```sh
# /home/abc/mysqldump -u root -p abc >abc.sql 
//在 home/abc 目录下使用 mysqldump 导出 abc 数据库的数据和表结构为 abc.sql 文件
```

#### 导入数据库
```shell
mysql -uroot -p database_name < database_name.sql
```
#### 使用 source 命令导入
使用 source 命令需要先登录到 MySQL 中，并创建一个空数据库：
```sql
mysql> create database abc;      # 创建数据库
mysql> use abc;                  # 使用已创建的数据库 
mysql> set names utf8;           # 设置编码
mysql> source /home/abc/abc.sql  # 导入备份数据库
```
>注意你备份的 SQL 文件路径。

### Wordpress 网站目录
>使用 tar 命令对目录完整打包
```sh
cd /var/www/html/
tar czf example.tar.gz example_folder
```
>在新服务器上按原目录解压即可

### Httpd 配置文件目录（按需）

>使用 tar 命令对 httpd 配置文件目录打包
```sh
cd /etc/
tar czf httpd.tar.gz httpd
```
>在新服务器上直接覆盖新 httpd 目录所有文件

