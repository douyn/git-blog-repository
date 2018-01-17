---
title: Android NDK开发流程
date: 2018年1月17日15:25:26
---
## Android NDK开发流程

使用ndk-build方式

* 配置环境变量
* 编写native代码
* 生成与native对应的头文件
* 利用头文件编写对应的C/C++代码
* 生成so库
* 使用so库

使用cmake方式

* 配置环境变量 //下载ndk,cmake,设置NDK_HOME
* 创建支持c/c++的项目 // 创建项目时选择c++support
* 创建c/c++文件,编写C/C++代码 
* 更改cmakelists.txt文件 // add_library/target_link_libraries改为自己的ndk name
* 在java代码中调用 

参考:

[cmake](http://www.jianshu.com/p/0261e6cceb3e)

[ndk-build](https://innofang.github.io/2017/04/16/Android-NDK%E5%BC%80%E5%8F%91%E4%BB%8E0%E5%88%B01/)