---
title: 应用获取系统权限
date: 2018年1月17日15:31:27
---
# 应用获取系统权限
将程序打包成系统应用才能获得系统权限
1. 添加清单文件
	
		android:sharedUserId="android.uid.system"
2. build未签名apk

	android studio --> build --> build apk(s)
3. 找到系统签名秘钥和系统签名工具

	系统密钥为：platform.pk8和platform.x509.pem
    
	AOSP路径： build\target\product\security
    
    工具为：signApk.jar
    
	AOSP路径：/out/host/linux-x86/framework/ signApk.jar
4. 对未签名apk进行签名
	
    使用下边的命令进行签名
    
    	java -jar $(signApk.jar的全路径地址) $(platform.x509.pem的全路径地址) $(platform.pk8的全路径地址) $(未签名apk文件的全路径地址) $(要生成的apk文件的全路径地址) 
	
5. 放入/system/app/文件夹下（需要root权限）

		adb remount
        adb push $(系统签名过的apk全路径地址) /system/app/
        
[签名工具和签名秘钥下载地址](http://pan.baidu.com/s/1nurbgYd)