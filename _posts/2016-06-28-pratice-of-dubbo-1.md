---
layout:     post
title:      "部署dubbo provider莫名错误:Error listenerStart之解决方法"
subtitle:   ""
date:       2016-06-28
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - tomcat
    - java
    - dubbo
description:  "部署dubbo provider错误:Error listenerStart之解决方法"    
---

### 部署dubbo provider错误:Error listenerStart之解决方法
*现象：*
tomcat7启动是报错：`严重: Error listenerStart`，无法注册服务

*排查：*
查看/var/log/tomcat7/localhost.2016-**-**.log文件，里面有详细的错误。

> 通常是文件权限不足造成的，默认的user.home没有写权限。

```java
严重: Exception sending context initialized event to listener instance of class com.zlfund.dubbo.context.ServiceContextLoader
java.lang.IllegalArgumentException: Invalid registry store file /usr/share/tomcat7/.dubbo/dubbo-registry-10.234.99.247.cache, cause:
Failed to create directory /usr/share/tomcat7/.dubbo!
        at com.alibaba.dubbo.registry.support.AbstractRegistry.<init>(AbstractRegistry.java:100)
        at com.alibaba.dubbo.registry.support.FailbackRegistry.<init>(FailbackRegistry.java:61)
        at com.alibaba.dubbo.registry.zookeeper.ZookeeperRegistry.<init>(ZookeeperRegistry.java:62)
``` 
 
*解决方法：*
chown tomcat7:tomcat7 /usr/share/tomcat7 搞定！


