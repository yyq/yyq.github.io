---
title: Java访问https网页时证书问题
author: young
layout: post
dsq_thread_id:
  - 3639564470
categories: [计算机,Infrastructure]
tags:
  - certification
  - https
  - java
  - ssl
date: 2015-03-30
---
报错信息：

<pre class="lang:java decode:true"><span style="color: #ff0000;">javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target</span></pre>

内部工具开发中，有一个步骤要去一个https的网页上抓取内容，然后该网站的证书没有被本机认可，于是乎拒绝了抓取内容。我本来的想法是，检测到证书问题之后，安装证书好了。上网搜了搜，解决方案大致两类：一是自己手动改改这里那里添加证书完事。二是在java代码中设定忽略所有证书问题。

如果是用第一种方案，那我的工具在所有其他工程师电脑上运行的时候，都需要它们自己去手动添加证书，那也太麻烦我的customer们了，虽然这种手动添加证书的事情只用干一次，但是他们都那么娇生惯养的，估计耽误他们工作时间来做这样的事情肯定不妥的。遂放弃的第一种思路，来看第二种，忽略所有，呵呵，代码能搞定。不过会有安全问题是吧。不过这又是内部工具，就把安全问题忽略好了。

<!--more-->

最后，我参考了这里，<a title="http://javaweb.org/?p=1237" href="http://javaweb.org/?p=1237" target="_blank">http://javaweb.org/?p=1237</a>解决了我的问题。

后记：我在想如果第一种方案也可以用代码自动搞定的话，那ssl的安全性有形同虚设？这像是一个悖论。如果是某安全限制能允许代码自动设置和绕过，那么此安全限制则形同虚设了。第二种方案，如果不是公司内部工具的话，我是绝对不会采用的，就算给出教程或者简单的文档提示，也会让用户自己去完成。

&nbsp;

需要在工程中添加这样一个class：
```
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;
 
import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.HttpsURLConnection;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSession;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;
 
public class SslUtils {
 
    private static void trustAllHttpsCertificates() throws Exception {
        TrustManager[] trustAllCerts = new TrustManager[1];
        TrustManager tm = new miTM();
        trustAllCerts[0] = tm;
        SSLContext sc = SSLContext.getInstance("SSL");
        sc.init(null, trustAllCerts, null);
        HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
    }
 
    static class miTM implements TrustManager,X509TrustManager {
        public X509Certificate[] getAcceptedIssuers() {
            return null;
        }
 
        public boolean isServerTrusted(X509Certificate[] certs) {
            return true;
        }
 
        public boolean isClientTrusted(X509Certificate[] certs) {
            return true;
        }
 
        public void checkServerTrusted(X509Certificate[] certs, String authType)
                throws CertificateException {
            return;
        }
 
        public void checkClientTrusted(X509Certificate[] certs, String authType)
                throws CertificateException {
            return;
        }
    }
     
    /**
     * 忽略HTTPS请求的SSL证书，必须在openConnection之前调用
     * @throws Exception
     */
    public static void ignoreSsl() throws Exception{
        HostnameVerifier hv = new HostnameVerifier() {
            public boolean verify(String urlHostName, SSLSession session) {
                System.out.println("Warning: URL Host: " + urlHostName + " vs. " + session.getPeerHost());
                return true;
            }
        };
        trustAllHttpsCertificates();
        HttpsURLConnection.setDefaultHostnameVerifier(hv);
    }
}

```

使用的example：

```
import java.io.OutputStreamWriter;
import java.net.URL;
import java.net.URLConnection;
import org.apache.commons.io.IOUtils;
 
public class SslTest {
     
    public String getRequest(String url,int timeOut) throws Exception{
        URL u = new URL(url);
        if("https".equalsIgnoreCase(u.getProtocol())){
            SslUtils.ignoreSsl();
        }
        URLConnection conn = u.openConnection();
        conn.setConnectTimeout(timeOut);
        conn.setReadTimeout(timeOut);
        return IOUtils.toString(conn.getInputStream());
    }
     
    public String postRequest(String urlAddress,String args,int timeOut) throws Exception{
        URL url = new URL(urlAddress);
        if("https".equalsIgnoreCase(url.getProtocol())){
            SslUtils.ignoreSsl();
        }
        URLConnection u = url.openConnection();
        u.setDoInput(true);
        u.setDoOutput(true);
        u.setConnectTimeout(timeOut);
        u.setReadTimeout(timeOut);
        OutputStreamWriter osw = new OutputStreamWriter(u.getOutputStream(), "UTF-8");
        osw.write(args);
        osw.flush();
        osw.close();
        u.getOutputStream();
        return IOUtils.toString(u.getInputStream());
    }
     
    public static void main(String[] args) {
        try {
            SslTest st = new SslTest();
            String a = st.getRequest("https://xxx.com/login.action", 3000);
            System.out.println(a);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
 
}

```