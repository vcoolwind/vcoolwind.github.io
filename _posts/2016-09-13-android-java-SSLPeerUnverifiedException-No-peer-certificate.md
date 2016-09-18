---
layout:     post
title:      "Android SSLPeerUnverifiedException问题解决"
subtitle:   ""
date:       2016-09-13
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - RabbitMQ
    - java
description:  "Android SSLPeerUnverifiedException问题解决"    
---
### Android SSLPeerUnverifiedException问题解决
{: .no_toc}

> 本文主要讲解下Android环境下 SSLPeerUnverifiedException异常的解决。

* 目录
{:toc}

### 问题现象
公司服务添加Ali WAF后，部分低版本Android https访问异常，debug显示如下信息：

```java
09-13 17:37:27.972: E/LoginActivity(30091): No peer certificate
09-13 17:37:28.735: W/System.err(30091): javax.net.ssl.SSLPeerUnverifiedException: No peer certificate
09-13 17:37:28.749: W/System.err(30091): 	at com.android.org.conscrypt.SSLNullSession.getPeerCertificates(SSLNullSession.java:104)
09-13 17:37:28.761: W/System.err(30091): 	at org.apache.http.conn.ssl.AbstractVerifier.verify(AbstractVerifier.java:93)
09-13 17:37:28.771: W/System.err(30091): 	at org.apache.http.conn.ssl.SSLSocketFactory.createSocket(SSLSocketFactory.java:388)
09-13 17:37:28.783: W/System.err(30091): 	at org.apache.http.impl.conn.DefaultClientConnectionOperator.openConnection(DefaultClientConnectionOperator.java:165)
09-13 17:37:28.793: W/System.err(30091): 	at org.apache.http.impl.conn.AbstractPoolEntry.open(AbstractPoolEntry.java:164)
09-13 17:37:28.804: W/System.err(30091): 	at org.apache.http.impl.conn.AbstractPooledConnAdapter.open(AbstractPooledConnAdapter.java:119)
09-13 17:37:28.815: W/System.err(30091): 	at org.apache.http.impl.client.DefaultRequestDirector.execute(DefaultRequestDirector.java:360)
09-13 17:37:28.825: W/System.err(30091): 	at org.apache.http.impl.client.AbstractHttpClient.execute(AbstractHttpClient.java:581)
09-13 17:37:28.834: W/System.err(30091): 	at org.apache.http.impl.client.AbstractHttpClient.execute(AbstractHttpClient.java:506)
09-13 17:37:28.844: W/System.err(30091): 	at org.apache.http.impl.client.AbstractHttpClient.execute(AbstractHttpClient.java:484)
```

### 问题分析
此类问题比较明显，应该是低版本的Android对TLS支持有问题，通过注册TLS解决问题。

### 问题解决
构建自定义的SSLSocketFactory并在发起请求前注册之。
发起调用片段如下：

```java
        HttpPost post = new HttpPost(SERVICE_URL);
        post.addHeader("http.protocol.content-charset", HTTP.UTF_8);
        List<NameValuePair> params = new ArrayList<NameValuePair>();
        String data = getMessage();
        params.add(new BasicNameValuePair("data", data));
        post.setEntity(new UrlEncodedFormEntity(params, HTTP.UTF_8));

        BasicHttpParams httpParams = new BasicHttpParams();
        HttpConnectionParams.setConnectionTimeout(httpParams, requestTimeout);
        HttpConnectionParams.setSoTimeout(httpParams, soTimeout);

        // 标记UA
        if (Constants.UA != null) {
            HttpProtocolParams.setUserAgent(httpParams, Constants.UA);
        }
        HttpClient client = new DefaultHttpClient(httpParams);
        HttpResponse response = null;
        
        if (SERVICE_URL.startsWith("https")) {
            // 设置SSL访问，解决证书问题。
            SSLContext ctx = SSLContext.getInstance("TLS");
            ctx.init(null, new TrustManager[] { new CustomX509TrustManager() }, new SecureRandom());
            SSLSocketFactory ssf = new CustomSSLSocketFactory(ctx);
            ssf.setHostnameVerifier(SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);
            ClientConnectionManager ccm = client.getConnectionManager();
            SchemeRegistry sr = ccm.getSchemeRegistry();
            sr.register(new Scheme("https", ssf, 443));
            DefaultHttpClient sslClient = new DefaultHttpClient(ccm, client.getParams());
            response = sslClient.execute(post);
        } else {
            response = client.execute(post);
        }
        // 处理响应
        if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
            HttpEntity entity = response.getEntity();
            String content = EntityUtils.toString(entity, "utf-8");
            return content;
        } else {
            String error = "网络出问题啦！请稍后重试！";
            throw new Exception("", error);
        }        
```
自定义类代码如下：

```java
import java.io.IOException;
import java.net.Socket;
import java.net.UnknownHostException;
import java.security.KeyManagementException;
import java.security.KeyStore;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.security.UnrecoverableKeyException;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

import javax.net.ssl.SSLContext;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;

import org.apache.http.conn.ssl.SSLSocketFactory;

/**
 * Taken from: http://janis.peisenieks.lv/en/76/english-making-an-ssl-connection-via-android/
 *
 */
public class CustomSSLSocketFactory extends SSLSocketFactory {
    SSLContext sslContext = SSLContext.getInstance("TLS");

    public CustomSSLSocketFactory(KeyStore truststore)
            throws NoSuchAlgorithmException, KeyManagementException,
            KeyStoreException, UnrecoverableKeyException {
        super(truststore);

        TrustManager tm = new CustomX509TrustManager();

        sslContext.init(null, new TrustManager[] { tm }, null);
    }

    public CustomSSLSocketFactory(SSLContext context)
            throws KeyManagementException, NoSuchAlgorithmException,
            KeyStoreException, UnrecoverableKeyException {
        super(null);
        sslContext = context;
    }

    @Override
    public Socket createSocket(Socket socket, String host, int port,
            boolean autoClose) throws IOException, UnknownHostException {
        return sslContext.getSocketFactory().createSocket(socket, host, port,
                autoClose);
    }

    @Override
    public Socket createSocket() throws IOException {
        return sslContext.getSocketFactory().createSocket();
    }
}

class CustomX509TrustManager implements X509TrustManager {

    @Override
    public void checkClientTrusted(X509Certificate[] chain, String authType)
            throws CertificateException {
    }

    @Override
    public void checkServerTrusted(java.security.cert.X509Certificate[] certs,
            String authType) throws CertificateException {

        // Here you can verify the servers certificate. (e.g. against one which is stored on mobile device)

        // InputStream inStream = null;
        // try {
        // inStream = MeaApplication.loadCertAsInputStream();
        // CertificateFactory cf = CertificateFactory.getInstance("X.509");
        // X509Certificate ca = (X509Certificate)
        // cf.generateCertificate(inStream);
        // inStream.close();
        //
        // for (X509Certificate cert : certs) {
        // // Verifing by public key
        // cert.verify(ca.getPublicKey());
        // }
        // } catch (Exception e) {
        // throw new IllegalArgumentException("Untrusted Certificate!");
        // } finally {
        // try {
        // inStream.close();
        // } catch (IOException e) {
        // e.printStackTrace();
        // }
        // }
    }

    public X509Certificate[] getAcceptedIssuers() {
        return null;
    }

}
```
### 参考
[这里](http://janis.peisenieks.lv/en/76/english-making-an-ssl-connection-via-android/)还有[这里](http://stackoverflow.com/questions/16719959/android-ssl-httpget-no-peer-certificate-error-or-connection-closed-by-peer-e)分析的很清楚了，感谢互联网让大家离的很近！


