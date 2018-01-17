---
title: Eclipse项目导入Android Studio注意事项
date: 2018年1月17日15:26:42
---

## Eclipse项目导入Android Studio注意事项:
1. 如果以前把项目从eclipse导入Android Studio失败过,最好要先删除掉gradle文件(包括build.gradle, gradle文件夹,以及gradlew, gradlew.bat文件)
2. 如果项目中使用了butterknife,需要删除某些文件,不然会报错:error: duplicate class: class_name$$ViewInjector$$.处理步骤如下:	1：删除eclipse项目中的.apt_generated文件夹和.factorypath文件 2：删除.classpath文件中的节点 3：重新导入到android studio中
3. android studio右下角的Gradle Console很重要,如果有gradle编译报错什么的,分析里边的日志基本上都可以得到答案.比如有的文件非法,你扔了一个.key文件到drawable文件夹中,在新版Android Studio中是会报错的,但是Messenges中的log可能是很片面的说检测到文件非法,但是在gradle console中可以看到具体哪一个文件出错,去修改.
4. 如果有.9图片,最好在这个项目级别的build.gradle文件的android{}中添加节点, 因为有很多时候在eclipse上能用的.9图片在as上不能用,如果不能用,是可以在gradle console中显示log的.Error:Execution failed for task ':app:mergeDebugResources'. > Some file crunching failed, see logs for details build gradle issues	
		aaptOptions {
            cruncherEnabled false
            useNewCruncher false
    	}
5. 如果用到google-play-service,导入到Android Studio后会自动添加远程依赖compile 'com.google.android.gms:play-services:+',但是最新的google-play-sercice使用到了appcompat-v7中的属性,可能与项目中的某些value值冲突,报值非法的错误,建议改为compile 'com.google.android.gms:play-services:7.0.0'
6. 去掉project.property中的项目依赖,单独去添加module依赖
7. 文件格式转换,可能运行时报非法字符的错误,只需要在右下角点击UTF-8,选择一个其他格式,弹出窗选择convert,完毕后再次点击XX(编码格式),选择utf-8,弹出窗选择convert,保存,重新编译.