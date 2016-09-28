---
layout:     post
title:      "对SEO友好的prerender简介"
subtitle:   ""
date:       2016-09-21
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - prerender
    - nodejs
    - SEO
description:  "对SEO友好的prerender简介"    
---
# 对SEO友好的Prerender简介
我们知道，前端工程师和SEOer会经常`打架`。目前的冲突：数据和页面分类是趋势，并且能大大提高开发效率；SEO对js加载的页面采集不够友好，使得决策者很难下决心全部使用前端js渲染数据。

有了Prerender，`麻麻`再也不用担心爬虫爬取了。

Prerender是一款非常强大的针对SEO友好的html渲染器，我曾经使用htmlunit实现了该功能，项目地址在[这里](https://github.com/vcoolwind/StaticCrawler)。
但整体效率上感觉比Prerender要低，该项目比较活跃，值得使用。

### 使用方法讨论
- 在nginx层配合lua，判断是爬虫或指定参数的，转交Prerender进行渲染；
- Prerender只负责渲染，不负责缓存（虽然，他可以这样做）；
- 遵循`谁使用谁缓存`的原则，在nginx可以在获取到Prerender进行缓存，下次访问时先查询缓存，缓存未命中时，再提交Prerender进行渲染。

### 安装Prerender
- 安装nodejs，参见[这里](https://vcoolwind.github.io/blog/2016/09/21/how-to-install-nodejs-on-debian-with-apt-get/)
- 安装并启用Prerender
```bash
$ git clone https://github.com/prerender/prerender.git
$ cd prerender
$ npm install
$ node server.js
```
- 测试-- 浏览器访问 http://localhost:3000/https://www.baidu.com 即可得到结果。

### 项目地址
- Prerender项目地址在[这里](https://github.com/prerender/prerender), enjoy it!
