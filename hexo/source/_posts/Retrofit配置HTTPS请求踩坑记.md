---
title: Retrofit Https踩坑记录
date: 2018年1月17日15:30:16
---
# Retrofit Https踩坑记录

## 前言 
新司机上路，坑多,本文重点是踩坑，不详细讲retrofit用法，本文不推荐使用信任所有证书的做法。

## 证书
分为多种格式, bks cer jks等，这里使用的是bks格式证书。

## BKS 做法
### 1.获取BKS证书，将证书放到项目raw目录下

#### 准备.cer文件
点击网站网址栏前的小锁按钮，选择详细信息，选择view certificate。
显示证书之后，点击详细信息，然后一直下一步，直到导出.cer文件。
![1](https://github.com/douyn/Photos/blob/master/res/20170301100223.png?raw=true)
![2](https://github.com/douyn/Photos/blob/master/res/20170301101115.png?raw=true)
![3](https://github.com/douyn/Photos/blob/master/res/20170301101333.png?raw=true)
![4](https://github.com/douyn/Photos/blob/master/res/20170301101625.png?raw=true)
![5](https://github.com/douyn/Photos/blob/master/res/20170301101705.png?raw=true)
![6](https://github.com/douyn/Photos/blob/master/res/20170301101806.png?raw=true)
![7](https://github.com/douyn/Photos/blob/master/res/20170301101638.png?raw=true)

#### 将.cer转换为.bks

[在Android应用中使用自定义证书,CER转BKS](http://blog.csdn.net/u010314594/article/details/50765534)

做法：1，下载特定版本的JCE Provider包

[http://pan.baidu.com/s/1c1ur13y](http://pan.baidu.com/s/1c1ur13y) 

or 

[http://www.bouncycastle.org/download/bcprov-jdk15on-146.jar](http://www.bouncycastle.org/download/bcprov-jdk15on-146.jar) (*现在连接失效*)

2，命令行输入以下命令
	
	keytool -importcert -v -trustcacerts -alias 位置1 \
	-file 位置2 \
	-keystore 位置3 -storetype BKS \
	-providerclass org.bouncycastle.jce.provider.BouncyCastleProvider \
	-providerpath 位置4 -storepass 位置5

位置1:是个随便取的别名 
位置2:cer或crt证书的全地址 
位置3:生成后bks文件的位置,建议写全地址 
位置4:上面下载JCE Provider包的位置 
位置5:生成后证书的密码。下边获取sslsocketfactory中会用到密码

以下例子:

	keytool -importcert -v -trustcacerts -alias xx -file E:\bks\xx.cer -keystore E:\bks\xx.bks -storetype BKS -providerclass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath E:\bks\bcprov-jdk15on-146.jar -storepass xxxxxx

成功之后会在你指定的位置生成bks文件.然后将文件放到项目raw目录下。


### 2.获取SSLSocketFactory
这里是https证书认证最关键的代码，一定要仔细查看。password和设置keystore的bks类型一定不要搞错。

	/**
     * 获取bks文件的sslsocketfactory
     * @param context
     * @return
     */
    public static SSLSocketFactory getSSLSocketFactory(Context context) {
        final String CLIENT_TRUST_PASSWORD = "123456";//信任证书密码，该证书默认密码是123456
        final String CLIENT_AGREEMENT = "TLS";//使用协议
        final String CLIENT_TRUST_KEYSTORE = "BKS";
        SSLContext sslContext = null;
        try {
            //取得SSL的SSLContext实例
            sslContext = SSLContext.getInstance(CLIENT_AGREEMENT);
            //取得TrustManagerFactory的X509密钥管理器实例
            TrustManagerFactory trustManager = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
            //取得BKS密库实例
            KeyStore tks = KeyStore.getInstance(CLIENT_TRUST_KEYSTORE);
            InputStream is = context.getResources().openRawResource(R.raw.traint);
            try {
                tks.load(is, CLIENT_TRUST_PASSWORD.toCharArray());
            } finally {
                is.close();
            }
            //初始化密钥管理器
            trustManager.init(tks);
            //初始化SSLContext
            sslContext.init(null, trustManager.getTrustManagers(), null);
        } catch (Exception e) {
            e.printStackTrace();
            Log.e("SslContextFactory", e.getMessage());
        }
        return sslContext.getSocketFactory();
    }

### 3.配置retrofit 
	String baseUrl = "https://skyish-test.yunext.com";
	int[] certificates = {R.raw.traint};
        String[] hostUrls = {baseUrl};
        OkHttpClient client = new okhttp3.OkHttpClient.Builder()
                .addInterceptor(new HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY))
                .sslSocketFactory(HTTPSUtils.getSSLSocketFactory(context))
				//.hostnameVerifier(HTTPSUtils.getHostNameVerifier(hostUrls)) 
                .readTimeout(10, TimeUnit.SECONDS)
                .connectTimeout(10, TimeUnit.SECONDS)
                .build();

        Retrofit retrofit = new Retrofit.Builder().baseUrl(baseUrl)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .client(client)
                .build();

配置好retrofit之后就可以使用了。

## 坑1：SSLContext is not initialized
	03-08 15:17:26.804 21672-21672/com.qiwo.enumlistdemo E/AndroidRuntime: FATAL EXCEPTION: main
	                                                                       Process: com.qiwo.enumlistdemo, PID: 21672
	                                                                       java.lang.RuntimeException: Unable to start activity ComponentInfo{com.qiwo.enumlistdemo/com.qiwo.enumlistdemo.RetrofitHttpsDemoActivity}: java.lang.IllegalStateException: SSLContext is not initialized.
	                                                                           at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2650)
	                                                                           at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2720)
	                                                                           at android.app.ActivityThread.-wrap12(ActivityThread.java)
	                                                                           at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1567)
	                                                                           at android.os.Handler.dispatchMessage(Handler.java:111)
	                                                                           at android.os.Looper.loop(Looper.java:207)
	                                                                           at android.app.ActivityThread.main(ActivityThread.java:5917)
	                                                                           at java.lang.reflect.Method.invoke(Native Method)
	                                                                           at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:789)
	                                                                           at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:679)
	                                                                        Caused by: java.lang.IllegalStateException: SSLContext is not initialized.
	                                                                           at com.android.org.conscrypt.OpenSSLContextImpl.engineGetSocketFactory(OpenSSLContextImpl.java:107)
	                                                                           at javax.net.ssl.SSLContext.getSocketFactory(SSLContext.java:358)
	                                                                           at com.qiwo.api.HTTPSUtils.getSSLSocketFactory(HTTPSUtils.java:158)
	                                                                           at com.qiwo.api.DemoHttpsApi.<init>(DemoHttpsApi.java:40)
	                                                                           at com.qiwo.enumlistdemo.RetrofitHttpsDemoActivity.initViewAndListener(RetrofitHttpsDemoActivity.java:37)
	                                                                           at com.doudou.common.base.BaseSwipeBackAppcompatActivity.onCreate(BaseSwipeBackAppcompatActivity.java:68)
	                                                                           at android.app.Activity.performCreate(Activity.java:6307)
	                                                                           at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1113)
	                                                                           at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2603)
	                                                                           at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2720) 
	                                                                           at android.app.ActivityThread.-wrap12(ActivityThread.java) 
	                                                                           at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1567) 
	                                                                           at android.os.Handler.dispatchMessage(Handler.java:111) 
	                                                                           at android.os.Looper.loop(Looper.java:207) 
	                                                                           at android.app.ActivityThread.main(ActivityThread.java:5917) 
	                                                                           at java.lang.reflect.Method.invoke(Native Method) 
	                                                                           at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:789) 
	                                                                           at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:679) 

### 原因：

1. 证书和证书密码不匹配。
2. 使用了错误的证书。证书类型不对。应该使用bks类型证书加载的确实cer类型的

### 解决方法：

CLIENT_TRUST_PASSWORD是证书的密码，必须与生成证书步骤里的设置的证书密码一致。如下：

	public static SSLSocketFactory getSSLSocketFactory(Context context) {
        final String CLIENT_TRUST_PASSWORD = "123456";//信任证书密码，该证书默认密码是changeit
        final String CLIENT_AGREEMENT = "TLS";//使用协议
        final String CLIENT_TRUST_KEYSTORE = "BKS";
        SSLContext sslContext = null;
		// ...
	}


如果是cer类型证书，需要使用生成bks方法重新生成bsk类型证书。

## 坑2：java.io.IOException: Hostname 'xx.com' was not verified

### 原因：

服务器主机名认证失败

### 解决方法：

#### 1. 如果okhttpclient中有hostnameverify的配置，加上一个自定义的HostNameVerify,如下

	((HttpsURLConnection) urlConnection).setHostnameVerifier(new HostnameVerifier() {
	  @Override
	  public boolean verify(String hostname, SSLSession session) {
	    return true;
	  }
	});

#### 2. 如果不需要HostNameVerify直接不设置就可以。	

	//.hostnameVerifier(HTTPSUtils.getHostNameVerifier(hostUrls)) 注释掉这句代码

## 坑3：javax.net.ssl.SSLPeerUnverifiedException

### 原因：

SSL链接时主机名验证失败

### 解决方法：

	//.hostnameVerifier(HTTPSUtils.getHostNameVerifier(hostUrls)) 注释掉这句代码

## 坑4：javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for cert

	javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for cert
    at com.android.org.conscrypt.OpenSSLSocketImpl.startHandshake(OpenSSLSocketImpl.java:333)
    at okhttp3.internal.io.RealConnection.connectTls(RealConnection.java:239)
    at okhttp3.internal.io.RealConnection.establishProtocol(RealConnection.java:196)
    at okhttp3.internal.io.RealConnection.buildConnection(RealConnection.java:171)
    at okhttp3.internal.io.RealConnection.connect(RealConnection.java:111)
    at okhttp3.internal.http.StreamAllocation.findConnection(StreamAllocation.java:187)
    at okhttp3.internal.http.StreamAllocation.findHealthyConnection(StreamAllocation.java:123)
    at okhttp3.internal.http.StreamAllocation.newStream(StreamAllocation.java:93)
    at okhttp3.internal.http.HttpEngine.connect(HttpEngine.java:296)
    at okhttp3.internal.http.HttpEngine.sendRequest(HttpEngine.java:248)
    at okhttp3.RealCall.getResponse(RealCall.java:243)
    at okhttp3.RealCall$ApplicationInterceptorChain.proceed(RealCall.java:201)
    at okhttp3.logging.HttpLoggingInterceptor.intercept(HttpLoggingInterceptor.java:212)
    at okhttp3.RealCall$ApplicationInterceptorChain.proceed(RealCall.java:190)
    at okhttp3.RealCall.getResponseWithInterceptorChain(RealCall.java:163)
    at okhttp3.RealCall.execute(RealCall.java:57)
    at retrofit2.OkHttpCall.execute(OkHttpCall.java:174)
    at retrofit2.adapter.rxjava.RxJavaCallAdapterFactory$RequestArbiter.request(RxJavaCallAdapterFactory.
    at rx.internal.operators.OperatorSubscribeOn$1$1$1.request(OperatorSubscribeOn.java:80)
    at rx.Subscriber.setProducer(Subscriber.java:211)
    at rx.internal.operators.OperatorSubscribeOn$1$1.setProducer(OperatorSubscribeOn.java:76)
    at rx.Subscriber.setProducer(Subscriber.java:205)
    at retrofit2.adapter.rxjava.RxJavaCallAdapterFactory$CallOnSubscribe.call(RxJavaCallAdapterFactory.ja
    at retrofit2.adapter.rxjava.RxJavaCallAdapterFactory$CallOnSubscribe.call(RxJavaCallAdapterFactory.ja
    at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:50)
    at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30)
    at rx.Observable.unsafeSubscribe(Observable.java:8666)
    at rx.internal.operators.OperatorSubscribeOn$1.call(OperatorSubscribeOn.java:94)
    at rx.internal.schedulers.CachedThreadScheduler$EventLoopWorker$1.call(CachedThreadScheduler.java:220
    at rx.internal.schedulers.ScheduledAction.run(ScheduledAction.java:55)
    at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:423)
    at java.util.concurrent.FutureTask.run(FutureTask.java:237)
    at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecut
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1113)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:588)
    at java.lang.Thread.run(Thread.java:818)
	Caused by: java.security.cert.CertificateException: java.security.cert.CertPathValidatorException: Trust 
    at com.android.org.conscrypt.TrustManagerImpl.checkTrusted(TrustManagerImpl.java:324)
    at com.android.org.conscrypt.TrustManagerImpl.checkServerTrusted(TrustManagerImpl.java:225)
    at com.android.org.conscrypt.Platform.checkServerTrusted(Platform.java:115)
    at com.android.org.conscrypt.OpenSSLSocketImpl.verifyCertificateChain(OpenSSLSocketImpl.java:571)
    at com.android.org.conscrypt.NativeCrypto.SSL_do_handshake(Native Method)
    at com.android.org.conscrypt.OpenSSLSocketImpl.startHandshake(OpenSSLSocketImpl.java:329)
	... 35 more
	Caused by: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found.

### 原因：

使用了错误的证书。证书验证失败。

### 解决

重新生成证书

## 后记
之前我是看的Tamic的做法，不能走通，不推荐使用它的那种做法。如果是使用它的那种做法，出现错误，请按照本文的做法，使用HTTPS。
## 参考

[http://www.cnblogs.com/lancer-ryn/p/5869696.html](http://www.cnblogs.com/lancer-ryn/p/5869696.html)

[http://www.jianshu.com/p/9a6c204616d2](http://www.jianshu.com/p/9a6c204616d2)