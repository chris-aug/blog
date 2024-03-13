---
title: Hexo 优化文章 URL
tags: Hexo
categories: Obsolete
abbrlink: 3223b82e
date: 2021-08-28 20:18:12
description: 
sticky: 
comments:
katex: 
aplayer:
hidden: true
---

## 如何优化 URL ?

在Hexo上，你可以将文章URL生成模式设置为自定义。
默认Hexo的文章通常以时间+文章标题生成URL，但是当你的文章标题为中文时，默认所生成的文章URL便会包含中文字符，包含中文字符的URL在使用中是需要转义的。当你在分享某些文章链接，在链接后面跟着长串的类似于这样的转义字符`%E4%B8%80%E8%B5%B7`，这便是包含中文字符URL被转义带来的问题，当然这只是其中之一。
自定义URL不但能解决掉这个问题，还有其他的优点：

- 便于 SEO
- 减小 URL 长度
- 取消文章标题与文章链接之间的关联
- 解决部分软件不能完整识别含中文字符URL，中文部分被自动省略导致到访问错误的页面

这样的优化方式同样适用与其他的博客，但是你可能需要参考你所使用的博客框架文档。

<!--more-->

## 修改永久链接
修改永久链接需要修改Hexo根目录下的`_config.yml`文件中的`permalink`字段。
默认情况下这一部分的值为`permalink: :year/:month/:title/`此时，文章的URL是由文章创建的年/月/文章标题组成。
```
permalink: :year/:month/:title/
permalink_defaults:
```
你需要将上面的内容修改成如下样式：
```
permalink: :year/:month/:urlname.html
permalink_defaults:
```
当然，你也可以在`permalink_defaults:`下增加默认值，写法为：
```
permalink_defaults:
	urlname: example
```
这样在你新创建文章的时候，可以通过Post Formatter配置文章的初始链接。
但是在你开始编辑文章的时候，还是要记得修改`urlname`字段。

## 修改文章模板
修改文章模板是为了让你每次创建文章的时候便创建 `urlname` 字段给你填写，也可以适用于前面的自动配置初始链接。
你需要修改博客根目录下 `scaffolds` 文件夹中的内容，为了方便你创建页面，这里便一起修改 `post.md` 和`page.md`.

```
post.md文件修改如下：
---
title: {{ title }}
date: {{ date }}
urlname: 
tags: 
categories: 
---

page.md文件修改如下：
---
title: {{ title }}
date: {{ date }}
urlname:
---
```
**注意：**
- 在字段的冒号后需填入一个空格，才能填入值。

现在开始在你创建完文章后便可以手动设置URL，参考你正在访问的这篇文章链接：
`https://blog.typeof.cc/2021/08/Hexo-optimized-article-url.html`，而不是`https://blog.typeof.cc/2021/08/Hexo优化文章URL.html`你可以尝试将此包含中文地址发送到 Wechat 等部分软件；你会发现中文部分被自动省略，要想被识别，必须将 URL 转义。

---

## 更新 2022/11/30
现在已经更新为采用 URL 自动生成：
- 插件 [hexo-abbrlink](https://github.com/rozbo/hexo-abbrlink)
- 本文当前 URL `https://blog.typeof.cc/posts/3223b82e.html`