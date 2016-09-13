---
layout:     post
title:      "ZooKeeper简易安装配置  "
subtitle:   ""
date:       2016-06-28
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - ZooKeeper
    - dubbo
description:  "ZooKeeper简易安装配置"    
---


### ZooKeeper简易安装配置
{: .no_toc}
> 本文以zookeeper 3.4.8版本部署到/opt/目录为例，讲解ZooKeeper的基本安装与配置。

* 目录
{:toc}

### 下载并部署[ZooKeeper-3.4.8](https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz)
	tar -xzf zookeeper-3.4.8.tar.gz /opt/

### 建立对应的应用目录
	mkdir -p /var/zookeeper/data
	mkdir -p /var/log/zookeeper

### 设置zoo.cfg配置文件
	#进入ZooKeeper配置目录，设置配置文件
	cd /opt/zookeeper-3.4.8/conf 
	cp zoo_sample.cfg  zoo.cfg
	vi zoo.cfg #注释全部，添加
		tickTime=2000
		initLimit=10
		syncLimit=5
		dataDir=/var/zookeeper/data
		clientPort=2181
		#第一台服务器地址
		server.1=10.234.99.247:2555:3555
		#第二台服务器地址
		server.2=10.234.99.248:2555:3555

### 配置服务器ID（服务器的id要和配置文件一致）
	#第一台服务器执行
	echo "1">/var/zookeeper/data/myid
	
	#第二台服务器执行
	echo "2">/var/zookeeper/data/myid

### 修改日志路径
	vi /opt/zookeeper-3.4.8/conf/log4j.properties： 
	改成 
	# Define some default values that can be overridden by system properties  
	zookeeper.root.logger=INFO,ROLLINGFILE  
	
	vi /opt/zookeeper-3.4.8/bin/zkEvn.sh
	首行添加：ZOO_LOG_DIR="/var/log/zookeeper"
	
	另外修改如下
	if [ "x${ZOO_LOG4J_PROP}" = "x" ]  
	then  
	    ZOO_LOG4J_PROP="INFO,ROLLINGFILE"  
	fi  

### 启动zookeeper  
	/opt/zookeeper-3.4.8/bin/zkServer.sh start
	

### 集群配置注意事项

> Zookeeper集群应不少于3台，尽量4~5台。

Zookeeper本身是集群，推荐配置不少于`3台`服务器。Zookeeper自身也要保证当一个节点宕机时，其他节点会继续提供服务。
- 如果是一个Follower宕机，还有2台服务器提供访问，因为Zookeeper上的数据是有多个副本的，数据并不会丢失，
- 如果是一个Leader宕机，Zookeeper会选举出新的Leader。
- 如果`只有一个`Zookeeper处于启动状态，那么他将`不再工作`。因为无法进行选举了，所以整个集群也就down了。

