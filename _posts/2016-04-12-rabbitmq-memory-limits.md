---
layout:     post
title:      "RabbitMQ内存限制异常排查与解决"
subtitle:   ""
date:       2016-06-28
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - RabbitMQ
description:  "RabbitMQ内存限制异常排查与解决"    
---


### RabbitMQ内存限制异常排查与解决
{: .no_toc}

> 本文主要讲解下在RabbitMQ使用过程中发现的内存限制导致的问题及排查过程 

* 目录
{:toc}

### 异常的感知
> 探测服务发现RabbiMQ服务异常（自定义探测服务，输出1表示RabbitMQ工作异常）
	
	[check]start...
	2016年 04月 12日 星期二 20:49:25 CST
	0
	[check]end.
	[check]start...
	2016年 04月 12日 星期二 20:49:28 CST
	1
	[check]end.
	[check]start...
	2016年 04月 12日 星期二 20:49:31 CST
	1
	[check]end.
	
### RabbitMQ异常的输出
	=INFO REPORT==== 12-Apr-2016::20:29:10 ===
	vm_memory_high_watermark set. Memory used:2554570744 allowed:2510541619
	
	=WARNING REPORT==== 12-Apr-2016::20:29:10 ===
	memory resource limit alarm set on node 'rabbit@ZL-RabbitMQ-Cluster2'.
	**********************************************************
	*** Publishers will be blocked until this alarm clears ***
	**********************************************************
	
### RabbitMQ的问题在哪里？
RabbitMQ服务器的内存的是6276354048，`select 2510541619.0/6276354048 = 0.4`，由此可以断定RabbitMQ的内存使用限制是40%。
查看RabbitMQ服务配置，没有指定内存使用。查阅[文档](https://www.rabbitmq.com/memory.html)，得知默认的内存限制是0.4。
> By default, when the RabbitMQ server uses above 40% of the installed RAM, it raises a memory alarm and blocks all connections. Once the memory alarm has cleared (e.g. due to the server paging messages to disk or delivering them to clients) normal service resumes.

由此可以得知，是RabbitMQ的内存超过了使用限制，从而停止了对外服务。
在阻塞外部服务前，RabbitMQ会启用持久化操作，尽量释放内存。
> Before the broker hits the high watermark and blocks publishers,
> it will attempt to free up memory by instructing queues to page their contents out to disc.
>  Both persistent and transient messages will be paged out (the persistent messages will already be on disc but will be evicted from memory).

### 参数分析

Key                                  | Documentation
------------------------------------ | ---------------------------------------------------------
vm_memory_high_watermark             | Memory threshold at which the flow control is triggered. See the memory-based flow control documentation.Default: 0.4
vm_memory_high_watermark_paging_ratio|Fraction of the high watermark limit at which queues start to page messages out to disc to free up memory. See the memory-based flow control documentation.Default: 0.5
heartbeat	                         |Value representing the heartbeat delay, in seconds, that the server sends in the connection.tune frame. If set to 0, heartbeats are disabled. Clients might not follow the server suggestion, see the AMQP reference for more details. Disabling heartbeats might improve performance in situations with a great number of connections, but might lead to connections dropping in the presence of network devices that close inactive connections.Default: 60 (580 prior to release 3.5.5)


- vm_memory_high_watermark：确定了内存使用比例或绝对值。
- vm_memory_high_watermark_paging_ratio：确定了何时执行消息从内存转移到硬盘。
- heartbeat:心跳探测时间，主动释放无效的链接。

其他配置看[这里](https://www.rabbitmq.com/configure.html).

```
配置样例分析：
[{rabbit, [{vm_memory_high_watermark_paging_ratio, 0.75},{vm_memory_high_watermark, 0.4}]}].
RabbitMQ使用40%的内存，在内存使用到达30%（0.75*0.4=0.3）时执行内存转移。
也可以把vm_memory_high_watermark_paging_ratio设置为大于1，这样等于说内存在限制前不转移。
```

> It is possible to set vm_memory_high_watermark_paging_ratio to a greater value than 1.0. 
> In this case queues will not page their contents to disc. 
> If this causes the memory alarm to go off, then producers will be blocked as explained above.

### 解决方案
- 加大使用内存，提高使用比例，缓解
- 服务器添加心跳机制
```
	[{rabbit, [{heartbeat, 600},
	           {vm_memory_high_watermark_paging_ratio, 0.85},
	           {vm_memory_high_watermark, 0.6}]
	}].
```
参考：[→ RabbitMQ + Memory Limits](http://stackoverflow.com/questions/12175156/rabbitmq-memory-limits)


		
