---
title: 【踩坑记录】 android 7.0 签名之INSTALL_PARSE_FAILED_NO_CERTIFICATES
date: 2018年1月17日15:23:42
---
# 【踩坑记录】 android 7.0 签名之INSTALL_PARSE_FAILED_NO_CERTIFICATES

## 前言
Android更新快，Android studio更新也快，Aradle更新更快

项目在之前的开发环境上没有一点问题（windows 10 / AS 2.2 / JDK 1.8 / gradle 2.2.0 / 设备HONOR 8(Android 7.0未安装app)），正好换电脑，换了一个开发环境（Ubuntu 16.04 / AS 2.3.1 / JDK 1.8 / gradle 2.3.1 / 设备有7.0也有7.0之前）。

导入项目之后，因为是最新的as版本，有一个更新提示
![罪魁祸首](https://github.com/douyn/Photos/blob/master/res/TIM%E5%9B%BE%E7%89%8720170410172524.jpg?raw=true)

google爸爸推荐更新能不更新吗，果断更新，long long ago之后，更新完了，终于俺的项目可以执行了，编译运行一条龙，木有任何问题，接下来就是给测试妹子打包，仍然是木有问题，打包出来之后，还本着不要打包出错的心态(丢人现眼)的心态安装了一次，perfect，木有问题。然后就等着所有bug都测试通过的喜讯了。然而，第一步就挂了。
## 现象
妹子说安装不上，心想这一定是你的打开方式不对，果断装起逼来，拿过来adb命令走起，果然啊，在我的手机上就可以，在测试的手机上就不行，通过adb报错发现INSTALL_PARSE_FAILED_NO_CERTIFICATES,这啥啊，不懂没关系，google baidu走起来。(网上大多数都是很久之前的过滤掉不要看，关键词不能少)

还有一个现象，打包的时候没有在意，也是一个关键失误。就是在gradle 2.3.1上打包的时候，最后一步要选择APK signature scheme，当时没在意直接选的v2 full apk,不要问为什么，英语好就是这么吊(脑洞翻译一下嘛v1打包jar，v2打包apk)。事实证明就是google给的提示不够明确啊。
## 原因
Android 7.0 引入一项新的应用签名方案 APK Signature Scheme v2，它能提供更快的应用安装时间和更多针对未授权 APK 文件更改的保护。在默认情况下，Android Studio 2.2 和 Android Plugin for Gradle 2.2 会使用 APK Signature Scheme v2 和传统签名方案来签署您的应用。

如这里所述，Android 7.0引入了新的签名方案V2。V2方案是对整个APK进行签名，而不是像V1一样只对JAR那样签名。如果您仅使用V2进行登录，并尝试在7.0之前的目标上安装，则会收到此错误，因为JAR本身未签名，并且7.0 之前的PackageManager无法检测V2 APK签名的存在。

## 解决方法
所以方案也就有两种

1. 降低gradle版本，像我们这样多人开发，每个人的studio版本都不一样，gradle版本也有不一样的，这样就会好一些。就是要把项目目录下的build.gradle中的gradle配置改一下。比如'classpath 'com.android.tools.build:gradle:2.3.1''改为'classpath 'com.android.tools.build:gradle:2.2.3''
2. 如果是个人的话，用越新的东西越能装逼，当然推荐用最新的gradle了，只是在打包的时候注意要兼容7.0以前和以后，这就需要注意把Signature Scheme V1和Signature Scheme V2都选上。

## 后记
不要害怕遇到问题，遇到问题就记下来装逼啊。
## 参考

[官方文档](https://developer.android.google.cn/aboutersionsugat/android-7.0.html#apk_signature_v2)

[解放方案1](http://stackoverflow.com/a/42377418)

[解决方案2](http://stackoverflow.com/a/43097991)