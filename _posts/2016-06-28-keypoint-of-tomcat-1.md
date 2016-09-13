---
layout:     post
title:      "Tomcat的那些坑之user.home"
subtitle:   ""
date:       2016-06-28
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - tomcat
    - debian
    - dubbo
description:  "Tomcat的那些坑之user.home"    
---

### Tomcat的那些坑之user.home使用
> 下文的X指的是对应的版本号，比如tomcat6、tomcat7、tomcat8

debian的tomcat安装后，在tomcat容器中获取user.*，比如 System.getProperty("user.home") ,获取的地址是/usr/share/tomcatX/。
 如果程序在运行过程中使用默认user.home目录进行读写，将会提示权限错误的异常，原因就是目录默认的所有者是root，而tomcat启动用户是tomcatX,无法正常读写user.home。

> 以dubbo举例，错误异常通常是：

```java
[27/06/16 07:45:09:009 CST] localhost-startStop-1 ERROR logger.Logger:  [DUBBO] Failed to init remote service reference at filed registry in class com.handu.open.dubbo.monitor.RegistryContainer, cause: Invalid registry store file /usr/share/tomcat7/.dubbo/dubbo-registry-
10.234.99.247.cache, cause: Failed to create directory /usr/share/tomcat7/.dubbo!, dubbo version: 2.5.3, current host: 127.0.0.1
java.lang.IllegalArgumentException: Invalid registry store file /usr/share/tomcat7/.dubbo/dubbo-registry-10.234.99.247.cache, cause: Failed to create directory /usr/share/tomcat7/.dubbo!
        at com.alibaba.dubbo.registry.support.AbstractRegistry.<init>(AbstractRegistry.java:100)
        at com.alibaba.dubbo.registry.support.FailbackRegistry.<init>(FailbackRegistry.java:61)
```

##### 解决方法：
- 对usr/share/tomcatX更改用户，`chown -R tomcatX:tomcatX /usr/share/tomcatX`
- 【推荐】程序不要依赖系统默认的`user.home`，设置为可配置或默认在启动类目录下。



