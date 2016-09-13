---
layout:     post
title:      "使用apt-get安装oracle-java"
subtitle:   ""
date:       2016-09-12
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - debian
    - java
description:  "使用apt-get安装oracle-java"    
---

### 使用apt-get安装oracle-java
{: .no_toc}

* 目录
{:toc}


### 设置apt-get 安装源，安装oracle-java7-installer
```bash
echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee /etc/apt/sources.list.d/webupd8team-java.list
echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886
apt-get update
apt-get install oracle-java7-installer	 
```


### 安装oracle-java7-installer后，会提示用wget开始下载jdk-7u80-linux-x64.tar.gz

    Downloading Oracle Java 7...
    --2016-06-24 09:12:47--  http://download.oracle.com/otn-pub/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz
    正在解析主机 download.oracle.com (download.oracle.com)... 208.31.254.10, 208.31.254.16
    正在连接 download.oracle.com (download.oracle.com)|208.31.254.10|:80... 已连接。
    已发出 HTTP 请求，正在等待回应... 302 Moved Temporarily
    位置：https://edelivery.oracle.com/otn-pub/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz [跟随至新的 URL]
    --2016-06-24 09:12:48--  https://edelivery.oracle.com/otn-pub/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz
    正在解析主机 edelivery.oracle.com (edelivery.oracle.com)... 184.50.90.127
    正在连接 edelivery.oracle.com (edelivery.oracle.com)|184.50.90.127|:443... 已连接。
    已发出 HTTP 请求，正在等待回应... 302 Moved Temporarily
    位置：https://edelivery.oracle.com/osdc-otn/otn-pub/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz [跟随至新的 URL]
    --2016-06-24 09:12:49--  https://edelivery.oracle.com/osdc-otn/otn-pub/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz
    再次使用存在的到 edelivery.oracle.com:443 的连接。
    已发出 HTTP 请求，正在等待回应... 302 Moved Temporarily
    位置：http://download.oracle.com/otn-pub/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz?AuthParam=1466730889_99c63151b7c6
    b21d3df6bef1ee97cec5 [跟随至新的 URL]--2016-06-24 09:12:49--  http://download.oracle.com/otn-pub/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz?AuthParam=1466
    730889_99c63151b7c6b21d3df6bef1ee97cec5正在连接 download.oracle.com (download.oracle.com)|208.31.254.10|:80... 已连接。
    已发出 HTTP 请求，正在等待回应... 200 OK
    长度：153530841 (146M) [application/x-gzip]
    正在保存至: “jdk-7u80-linux-x64.tar.gz”
         0K ........ ........ .
    
> 如果下载很慢，你可以使用迅雷等工具下载对应文件，放入/var/cache/oracle-jdk7-installer/jdk-7u80-linux-x64.tar.gz。详细操作见文末。



### 设置java7环境变量
```bash
apt-get install oracle-java7-set-default
```
    
> To automatically set up the Java 7 environment variables, you can install the following package: 
> If you've already installed oracle-java6-set-default or oracle-java8-set-default, they will be automatically removed when installing oracle-java7-set-default (and the environment variables will be set for Oracle Java 7 instead).


### 设置default-java便于其他程序使用
{% highlight bash %}	
rm /usr/lib/jvm/default-java 
ln -s /usr/lib/jvm/java-7-oracle /usr/lib/jvm/default-java
{% endhighlight %}

### 安装成功！
```bash
java -version
#(启动tomcat后检查是否使用的是对应版本的java程序  /usr/lib/jvm/java-*-oracle)
ps -er|grep java  
```

参考：[how-to-install-oracle-java-7-in-debian](http://www.webupd8.org/2012/06/how-to-install-oracle-java-7-in-debian.html)

       
        

***
---

### 默认下载慢的解决方法

-  手工终止下载

    > ps -ef \| grep wget
    
    ``` 
    root 26355  26348  0 09:12 pts/1    00:00:00 wget --continue --no-check -certificate -O jdk-7u80-linux-x64.tar.gz --header Cookie: oraclelicense=a http://download.oracle.com/otn-pub/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz
    ```
    
    > killall wget
    
    
- 获取到对应的下载地址
    > http://download.oracle.com/otn-pub/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz

- 使用下载工具下载对应的文件后放入目标服务器  
    > cp jdk-7u80-linux-x64.tar.gz /var/cache/oracle-jdk7-installer/

- 重新进行安装  apt-get install oracle-java7-installer