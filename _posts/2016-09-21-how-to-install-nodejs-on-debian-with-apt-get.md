---
layout:     post
title:      "如何在debian上使用apt-get安装nodejs"
subtitle:   ""
date:       2016-09-21
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - debian
    - nodejs
description:  "如何在debian上使用apt-get安装nodejs"    
---
# 如何在debian上使用apt-get安装nodejs

### 查看操作系统版本

```
root@bogon:/tmp# cat /etc/debian_version 
8.6
```

###  设置更新源

```bash
apt-get install curl
curl -sL https://deb.nodesource.com/setup | bash -
```

### 安装 nodejs

```bash
apt-get install -y nodejs
```
