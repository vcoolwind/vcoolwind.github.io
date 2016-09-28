---
layout:     post
title:      "gitlab配置指南"
subtitle:   "gitlab安装、配置、备份、恢复、迁移"
date:       2016-09-28
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - prerender
    - nodejs
    - SEO
description:  "gitlab安装、配置、备份、恢复"    
---
# gitlab安装、配置、备份、恢复
本文详细介绍gitlab配置常用的动作，关注安装、配置、备份与恢复等运维功能。

* 目录
{:toc}

### 安装
```bash
-  wget https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh 
-  bash script.deb.sh
-  修改 /etc/apt/sources.list.d/gitlab_gitlab-ce.list，使用清华的镜像：
	#deb https://packages.gitlab.com/gitlab/gitlab-ce/debian/ jessie main
	#deb-src https://packages.gitlab.com/gitlab/gitlab-ce/debian/ jessie main
	deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/debian jessie main
-  apt-get update 
-  apt-get install gitlab-ce  
```
顺便介绍一下，如何指定版本号安装
```bash
# 查询可支持的版本
apt-cache showpkg gitlab-ce
#安装指定版本  
apt-get install gitlab-ce=8.10.5-ce.0
```

### 配置(支持163.com)
修改 /etc/gitlab/gitlab.rb ，设置发送邮件
```
external_url 'http://192.168.0.60'

gitlab_rails['gitlab_email_from'] = 'XXX@163.com'
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "XXX"
gitlab_rails['smtp_password'] = "YYY"
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = :login
gitlab_rails['smtp_enable_starttls_auto'] = false
```

### 启动
使用reconfigure启动gitlab
```bash
gitlab-ctl reconfigure
```

### 日常的版本管理
然后就可以登录gitlab进行日常的版本管理了，gitlab使用方法比较简单，web方式的管理简洁易懂，不再赘述。
顺便讲解一下，git仓库的迁移，这个通常会用到。
```bash
git push --mirror git@192.168.85.139:gxx/pxx.git
```

### gitlab备份
#### 一键备份
使用一条命令即可创建完整的Gitlab备份:
```bash
gitlab-rake gitlab:backup:create
```
使用以上命令会在/var/opt/gitlab/backups目录下创建一个名称类似为1475050193_gitlab_backup.tar的压缩包, 这个压缩包就是Gitlab整个的完整部分, 其中开头的1475050193是备份创建的日期.

#### 修改备份目录
修改/etc/gitlab/gitlab.rb来修改默认存放备份文件的目录:
```
gitlab_rails['backup_path'] = '/mnt/backups'
```
/mnt/backups修改为你想存放备份的目录即可, 修改完成之后使用`gitlab-ctl reconfigure`命令重载配置文件即可

#### 自动备份
可以通过`crontab`使用备份命令实现自动备份:
```bash
crontab -e
#加入以下, 实现每天凌晨2点进行一次自动备份:
0 2 * * * /usr/bin/gitlab-rake  gitlab:backup:create
```

### gitlab恢复
```bash
# 停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq

# 从1393513186编号备份中恢复
gitlab-rake gitlab:backup:restore BACKUP=1475050193

# 启动Gitlab
gitlab-ctl start
```

### gitlab迁移
将备份文件拷贝到新服务器上的/var/opt/gitlab/backups即可(如果你没修改过默认备份目录的话). 但是需要注意的是新服务器上的Gitlab的版本必须与创建备份时的Gitlab版本号相同，可以通过在新服务指定版本安装gitlab来解决。







 