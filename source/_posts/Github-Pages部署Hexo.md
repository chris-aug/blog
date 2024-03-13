---
title: Github-Pages 部署 Hexo
tags: Hexo
categories: Tips
cover: /img/cover/5ffdd8dd.webp
abbrlink: 5ffdd8dd
date: 2021-08-27 20:21:07
description: Hexo 入门指北
sticky: 
comments:
katex: 
aplayer:
---
![封面图](/img/cover/5ffdd8dd.webp)

## 什么是 Hexo

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](https://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

## 什么是 Github Pages
GitHub Pages 是一个静态站点托管服务。
Github 页面旨在直接从 GitHub 仓库中直接托管您的个人、组织或项目页面。了解关于 GitHub Pages 网站不同类型的更多信息，请参阅[用户、组织和项目页面](https://help.github.com/articles/user-organization-and-project-pages/)。
GitHub Pages 是静态站点托管服务器，不支持 PHP ，Ruby 或 Python 等服务器端代码。

## 搭建步骤：

- Github 创建个人仓库
- 安装 Git
- 安装 Node.js
- 安装 Hexo
- Hexo 基础命令
- 推送网站
- 绑定域名
- 更换主题
- 初识 MarkDown 语法
- 发布文章
- 个性化设置

### 创建 Github 个人仓库
登录到Github创建一个新的仓库，仓库名称为 username.github.io。这是固定写法，免费账户 Github Pages 子域名必须为用户名；如果此时你还没有账户你需要自行在 [Github](https://github.com) 注册。

### 安装 Git
什么是 Git? 简单来说 Git 是开源的分布式版本控制系统，用于敏捷高效地处理项目。我们网站在本地搭建好了，需要使用 Git 同步到 GitHub 上。如需更详细的信息，请查看 [Git 官方文档](https://git-scm.com/doc)或者其他的教程类文章。
- Windows：下载并安装 [Git](https://git-scm.com/download/win).
- Mac：使用 [MacPorts](https://www.macports.org/) 或者下载 [安装程序](https://sourceforge.net/projects/git-osx-installer/)，当然最好的方式是安装Xcode。
- Linux：都用 Linux 了，我相信你是会的。

### 安装 Node.js
Node.js 为大多数平台提供了[官方安装程序](https://nodejs.org/en/download/)。
与 Hexo 官方文档的不同，我推荐你使用 LTS 版本的 Node.js.

- Windows：如果你的需求仅在使用 Hexo，那么建议直接使用[官方安装程序](https://nodejs.org/en/download/)。
- Mac：使用 [MacPorts](https://www.macports.org/) 或者下载[安装程序](https://nodejs.org/en/download/)。
- Linux：我相信你

> 在 Windows 下 npm 会和 Node.js 一起安装，Linux 下需要手动安装 npm.

## 安装 Hexo
### 使用 npm 命令安装 Hexo
输入：
```sh
npm install -g hexo-cli
```
### 初始化 Hexo
```
hexo init Blog #Blog为目录名称可带路径自定义
```
安装完成后使用 cd 命令进入目录`cd Blog`
此时可以新建一个文档进行测试也可以直接启动 Hexo Server.
Hexo 在本地是实时文件渲染所有设置或文档更改直接刷新即可，不需要执行 Generate.

## Hexo 基础命令
```sh
npm install -g hexo #安装Hexo
npm update -g hexo  #更新
hexo init #初始化hexo

hexo n "New Doc" == hexo new "New Doc" #新建文章
hexo g == hexo generate #生成文件
hexo s == hexo server   #启动本地服务
hexo d == hexo deploy   #部署
hexo c == hexo clean    #清理生成的本地文件
```
## 推送网站
### 配置 Git
```
git config --global user.name "Github Username"
git config --global user.email "Github注册邮箱"
```

### 创建 Personal Access Token (either)
Github 常规操作的在2021年情人节（8月14日）全局干掉了 repo 使用密码验证，所以不论是 HTTPS 还是 SSH 从现在开始你都不能使用密码，只能在 Personal Access Token 或者 SSH 密钥中二选一。
[查看官方解释](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/)
创建 Token：
1、在任何页面的右上角，点击个人资料照片，然后点击 Settings
![userbar-account-settings](https://docs.github.com/assets/images/help/settings/userbar-account-settings.png)
2、在左侧边栏中，点击 Developer Settings
![developer-settings](https://docs.github.com/assets/images/help/settings/developer-settings.png)
3、在左侧边栏中，单击 Personal access tokens
![personal_access_tokens_tab](https://docs.github.com/assets/images/help/settings/personal_access_tokens_tab.png)
4、创建 Token 并设置一个描述名称和有效期以及控制权限。

5、和 SSH 密钥一样，同样有方法可以避免每次都要输入 Token，你可以将 Token 写入远程仓库链接

```sh
git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git
```
- `<your_token>`：换成你自己得到的 token
- `<USERNAME>`：是你自己 github 的用户名
- `<REPO>`：是你的仓库名称

### 生成 SSH 密钥文件 (either)
生成密钥文件是便于推送的时候解决身份验证问题
创建密钥：
```
ssh-keygen -t rsa -C "Github 注册邮箱"
```
可以不设置密码，直接三个回车，也可以根据个人需求再次设置密码。完成后转到.ssh文件夹中找到id_rsa.pub文件，使用记事本打开将内容复制。
打开 [Github_Settings_Keys](https://github.com/settings/keys)，绑定公钥。其中Title任意填写，将 id_rsa.pub中复制出来的文件粘贴进Key点击Add SSH Key 即可。
![Add SSH Key](https://pic1.zhimg.com/80/v2-72a3f22c080e99343c3cc4aabce10e3c_1440w.jpg)


测试密钥访问：
```sh
ssh -T git@github.com
#输入yes后看到返回值为:
Hi "Github Username" You've successfully authenticated, but Github does not provide shell access.
#即验证成功
```
### 准备推送
打开网站根目录编辑 ` _config.yml`  文件
查看文件末尾d eploy 部分
参考如下：

```
deploy: 
	type: git
	repo: git@github.com:username/username.github.io
	#如果在上一步中你选择了创建密钥那么你应该填写如上所述的SSH地址
	#如果你选择的是创建Token那么你应该将地址写为https地址
	#e.g repo: https://github.com/username/username.github.io
	branch: gh-pages #推送位置，可根据规则自定义
```
### 生成文件与推送
在完成了上述操作后，现在需要加载一个推送插件，命令为：
```sh
npm install hexo-deployer-git --save
```
现在，你可以生成页面文件，并部署到 Github Pages.
```
hexo clean && hexo generate && hexo deploy
#当然，你也可以三条命令分开执行或者简写
```
在此之前你也可以使用 `hexo server `使用本地预览确认是否符合你所需要。
现在，你已经可以打开你的仓库或者 username.github.io 查看线上预览了。

## 绑定域名
第一步当然你得拥有一个域名，但是我并不想在此处详细描写这个步骤。
### 设置 DNS 解析
第二步便是设置解析，找到你的域名 DNS 解析控制台
推荐记录如下：

```
$ dig WWW.EXAMPLE.COM +nostats +nocomments +nocmd
    > ;WWW.EXAMPLE.COM.          IN    A
    > WWW.EXAMPLE.COM.   3592    IN    CNAME   YOUR-USERNAME.github.io.
    > EXAMPLE.COM.         3592    IN    CNAME   YOUR-USERNAME.github.io.
```
简单地说，你需要将设置一个 WWW 的 CNAME 记录和一个 @ 的 CNAME 记录，指向地址为 YOUR-USERNAME.github.io.
### 创建 CNAME 文件
既然已经完成了解析设置，现在需要在你的 Pages 仓库文件根目录下创建一个名为 CNAME 的文件
但是，每一次 Hexo 的推送都会覆盖掉之前的文件，所以在这里，你需要将 CNAME 文件存放于本地目录的 source 目录下，这样在你推送网站时，这个文件会一起推送，而不会因为在线创建而被推送所覆盖。
 CNAME 文件中所需要填写的内容非常简单 example.com 不需要其他的，填入你的主域即可。

### 访问你的域名
等待你的 DNS 解析生效后，便可以使用 example.com 访问

### 更换主题
选取你喜欢的主题，使用 `git clone` 命令将源码下载至 themes 目录，在主配置文件中将 themes 字段值修改为主题名称即可；其他设置你需要参考主题对应的文档。

## 初识 MarkDown 语法
请查看这份[中文文档](https://www.appinn.com/markdown/)。
关于 MarkDown 编辑器，推荐 [Typora](https://typora.io/).

## 发布新文章
使用命令创建新的文章，此时便会生成一个文章名称 .md 的文件
```
hexo new "新文章名称"
```
你需要使用支持 MarkDown 格式的文本编辑器来编辑，推荐请查看上一节。

### 个性化设置
网站的配置文件为根目录下的 `_config.yml` 文件，按需修改。
部分主题也在主题目录下同样提供了一个 `_config.yml` 文件，修改这个你需要参考主题文档。