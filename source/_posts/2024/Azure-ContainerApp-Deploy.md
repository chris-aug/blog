---
title: Azure ContainerApp 部署应用与持久化数据存储
tags:
  - ChatGPT
  - Azure
  - PaaS
  - Docker
  - Kubernetes
  - Cloud Service
categories: Container
abbrlink: 619
date: 2024-03-28 11:22:07
description:
cover:
sticky:
comments:
katex:
aplayer:
hidden:
---

# ContainerApp
以下是 Azure 官方对 ContainerApp 的定义：
> Azure 容器应用是一个无服务器平台，可让你在运行容器化应用程序时维护较少的基础结构并节省成本。 容器应用提供所有最新的服务器资源，以确保你的应用程序稳定和安全，而无需担心服务器配置、容器编排和部署详细信息。
> Azure 容器应用的常见用途包括：
>- 部署 API 终结点
>- 承载后台处理作业
>- 处理事件驱动的处理
>- 运行微服务

<!--more-->
---

# 容器应用部署
## 创建容器应用环境
> ContainerApp Environmnet

- 搜索容器应用 (ContainerApp)
- 进入并选择创建容器应用
	- 基本信息选项卡 
		- 订阅 - 自行填写
		- 资源组 - 自行填写
		- 容器应用名称 - 自行填写
		- 容器应用环境 - 点击新建
	- 创建容器应用环境
		- 容器应用环境名称
		- 根据需求选择环境类型，低负载可选仅消耗
		- 冗余选项，可以选择禁用，但选择在工作负载模式运行时考虑启用
{% img /img/posts/2024/03/Azure-ContainerApp-Deploy/create-container-envManage.png 600 500 创建容器应用环境 %}

## 创建容器
### 容器选项卡
- 名称 - 自行命名
- 映像源 - 选择 DockerHub 或其他注册表
- 映像类型 - 根据实际情况选择
- 命令重写 - 根据需求
- 容器资源分配 - CPU 和内存 - 根据需求
- 环境变量 - 按实际情况写
{% img /img/posts/2024/03/Azure-ContainerApp-Deploy/create-container.png 605 595 创建容器 %}
### Ingress 
- 流入量 - 启用(入站请求)
- Ingress 流量 - 接受来自任何位置的流量
- 入口类型 - 按实际选择 HTTP 或 TCP (大多数为 HTTP)
- 客户端证书模式 - 可忽略
- 传输 - 自动
- 目标端口 - 输入容器服务中暴露的端口
- 会话亲和性和其他 TCP 端口 - 按实际需求
- 标记、审阅+创建 - 没有需要的话直接创建即可
{% img /img/posts/2024/03/Azure-ContainerApp-Deploy/Ingress.png 600 560 Ingress %}

---
# 持久化数据
{% note info %}
**此处使用 Azure 存储账户和文件共享功能实现**
存储账户和文件共享的创建查看：
1. [创建存储帐户 - Azure Storage | Microsoft Learn](https://learn.microsoft.com/zh-cn/azure/storage/common/storage-account-create?tabs=azure-portal)
2. [有关创建和使用 Azure 文件共享的快速入门 | Microsoft Learn](https://learn.microsoft.com/zh-cn/azure/storage/files/storage-how-to-use-files-portal?tabs=azure-portal)
{% endnote %}
## 容器应用环境添加文件共享
- 打开需要操作的容器应用环境
- 找到 Azure 文件共享
- 点击添加
	- 名称 - 自行命名
	- 存储账户 - 前往存储账户/访问密钥查看
	- 存储账户密钥 - 前往存储账户/访问密钥查看
	- 共享名 - 前往存储账户/文件共享查看
{% img /img/posts/2024/03/Azure-ContainerApp-Deploy/container-env-add-file-share.png 600 350 Ingress %}
## 挂载文件共享
> 简单的记录一下 Portal 的操作步骤，Azure 文档目前只提供了 Azure-CLI 操作步骤
> 参考：[在 Azure 容器应用中创建 Azure 文件存储卷装载 | Microsoft Learn](https://learn.microsoft.com/zh-cn/azure/container-apps/storage-mounts-azure-files?tabs=bash)
- 进入对应容器应用页面
- 应用程序 - 容器 - 启用编辑和部署
{% img /img/posts/2024/03/Azure-ContainerApp-Deploy/container-app.png 600 290 ContainerApp Page %}
- Volumes 页面 - 添加存储
	- Volume Type - Azure file volmue
	- 名称 - 自行填写 - 下一步要用
	- 文件共享 - 选择你在容器应用环境中添加的文件共享
	- MountOption（重点）- 填入 `uid=1000,gid=1000,nobrl,mfsymlinks,cache=none`

{% img /img/posts/2024/03/Azure-ContainerApp-Deploy/add-volume.png 600 380 Add Volume %}

- 容器页面 - 选中并编辑容器
- 装载卷
	- 卷名称 - 上一步添加存储时自行填写的名称
	- Mount Path - 需要持久化存储的数据目录
- 保存操作 - 创建修订
{% img /img/posts/2024/03/Azure-ContainerApp-Deploy/edit-container.png 600 550 Edit Container %}

---
# 结尾
- 我部署了什么？
	-  [OneAPI OpenAI 接口管理 & 分发系统](https://github.com/songquanpeng/one-api)
- 用途和需求
	- 主要自用，用量不大
	- 功能上主要是 API 请求转发和鉴权
	- 定价灵活，支持自动扩缩容，最好是能缩到 0
	- 对于容器能提供持久化数据单独存储，而且数据是能导出
- 为什么需要缩到 0
	- 每天用的时间去除中断，也就几个小时，而且不扩容的情况下，并发越高越 vCPU 时间单价相对请求数的价格比就越便宜。我自己的使用通常都是用于翻译论文、市场报告、书籍、文章等是属于是时间集中，并发请求量大的用法；用完了就需要自动缩到 0 停止计费节省费用
- 为什么选择 Azure ContainerApp?
	- 免费的 vCPU 和 内存时间已经足够我使用了，在缩到 0 时不计费也不计算免费额度
- 如果只实现 API 转发和鉴权，为什么不用 Works?
	- Cloudflare Works 目前来说最大的问题是用的是滥用的人太多了，大量的搭建网站反向代理，利用 WebSocket 的支持转发网络请求等等；包括 CDN 部分的滥用，不久前这个手已经伸到了 AWS CloudFront，因为 AWS 也支持 WebSocket，同时提供了每个月 1TB 的免费 CDN 流量
- 为什么不是 Zeabur, Railway, Heroku, SealOS 等等？
	- Zeabur, Railway, Heroku 这些部署 Container 都有最低消费
	- SealOS 目前做不到缩到 0, 持久化数据存储好像是支持的，以后有需求再尝试