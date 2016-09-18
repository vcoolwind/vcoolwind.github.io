---
layout:     post
title:      "Java 忽略SSL认证通用代码"
subtitle:   ""
date:       2016-09-13
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - java
    - SSL
description:  "Java 忽略SSL认证通用代码"    
---
### Java 忽略SSL认证通用代码
> Java SSL访问某些网站时，经常会出现SSL认证相关的一系列问题，本文给出一套通用代码，忽略证书交互。由于问题比较明显，直接上代码。

调用方法 -- 在http调用前的系统入口处执行如下语句：

```java
static {
        Protocol https = new Protocol("https", new HTTPSSecureProtocolSocketFactory(), 443);
        Protocol.registerProtocol("https", https);
}
```

自定义类HTTPSSecureProtocolSocketFactory代码如下：

```java
import java.io.IOException;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.Socket;
import java.net.SocketAddress;
import java.net.UnknownHostException;
import java.security.KeyManagementException;
import java.security.NoSuchAlgorithmException;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

import javax.net.SocketFactory;
import javax.net.ssl.SSLContext;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;

import org.apache.commons.httpclient.ConnectTimeoutException;
import org.apache.commons.httpclient.params.HttpConnectionParams;
import org.apache.commons.httpclient.protocol.ProtocolSocketFactory;

/**
* <p>Title: HTTPSSecureProtocolSocketFactory</p>
* <p>Description: 匿名认证，可以考虑使用 。  在调用处声明 static {
        Protocol https = new Protocol("https", new HTTPSSecureProtocolSocketFactory(), 443);
        Protocol.registerProtocol("https", https);
    } </p>
* <p>Company: zlfund</p> 
* @author    wangyanfeng
* @date       2015年4月8日
*/
public class HTTPSSecureProtocolSocketFactory implements ProtocolSocketFactory {// SecureProtocolSocketFactory
    private SSLContext sslcontext = null;

    private SSLContext createSSLContext() {
        SSLContext sslcontext = null;
        try {
            sslcontext = SSLContext.getInstance("SSL");
            sslcontext.init(null, new TrustManager[] { new TrustAnyTrustManager() }, new java.security.SecureRandom());
        } catch(NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch(KeyManagementException e) {
            e.printStackTrace();
        }
        return sslcontext;
    }

    private SSLContext getSSLContext() {
        if (null == this.sslcontext) {
            this.sslcontext = createSSLContext();
        }
        return this.sslcontext;
    }

    public Socket createSocket(Socket socket, String host, int port, boolean autoClose) throws IOException,
            UnknownHostException {
        return getSSLContext().getSocketFactory().createSocket(socket, host, port, autoClose);
    }

    public Socket createSocket(String host, int port) throws IOException, UnknownHostException {
        return getSSLContext().getSocketFactory().createSocket(host, port);
    }

    public Socket createSocket(String host, int port, InetAddress clientHost, int clientPort) throws IOException,
            UnknownHostException {
        return getSSLContext().getSocketFactory().createSocket(host, port, clientHost, clientPort);
    }

    public Socket createSocket(String host, int port, InetAddress localAddress, int localPort,
            HttpConnectionParams params) throws IOException, UnknownHostException, ConnectTimeoutException {
        if (params == null) {
            throw new IllegalArgumentException("Parameters may not be null");
        }
        int timeout = params.getConnectionTimeout();
        SocketFactory socketfactory = getSSLContext().getSocketFactory();
        if (timeout == 0) {
            return socketfactory.createSocket(host, port, localAddress, localPort);
        } else {
            Socket socket = socketfactory.createSocket();
            SocketAddress localaddr = new InetSocketAddress(localAddress, localPort);
            SocketAddress remoteaddr = new InetSocketAddress(host, port);
            socket.bind(localaddr);
            socket.connect(remoteaddr, timeout);
            return socket;
        }
    }

    private static class TrustAnyTrustManager implements X509TrustManager {
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
        }

        public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
        }

        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[] {};
        }
    }

}
```
