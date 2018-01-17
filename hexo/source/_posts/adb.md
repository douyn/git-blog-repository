---
title: ADB总结
date: 2018年1月17日15:24:50
---
## adb总结

>什么是adb，c/s架构，adb命令分为3种

adb命令是程序自带的一些命令，adb shell是调用Android系统中的命令，这些系统是放在system/bin目录下

#### adb命令

###### adb devices

###### *adb get-state:device,offline,unknown*

###### adb kill-server,adb start-srevre

###### *adb bugreport 打印dumpsys,dumpstate,logcat的输出，分析错误,可以和logcat一样重定向到文件中*

###### adb pull,adb push

###### adb install, adb uninstall

###### *adb root, adb remount* 获取 root 权限，并挂载系统文件系统为可读写状态

###### *adb reboot* bootloader , 重启设备，进入 fastboot 模式，同 adb reboot-bootloader 命令

###### recovery , 重启设备，进入 recovery 模式，经常刷机的同学比较熟悉这个模式

###### *adb forward* 将 宿主机上的某个端口重定向到设备的某个端口

###### *adb connect* 连接远程Android设备

// 根据TAG和级别过滤日志输出

###### *adb logcat* 
adb logcat [-s] [ClassName:[PREVISOUS]] [*:[PREVIOUS]]

	adb logcat // 直接输出的终端
	adb logcat > c:/log.txt // 保存到文件
    adb logcat ActivityManager:I PowerManager:D *:S
    adb logcat *:W // 显示所有优先级大于等于“warning”的日志
 	adb logcat -s ActivityManager
logcat命令列表:

	-d 将日志显示到控制台后退出
    -c 清理已经存在的目录
    -f <filename> 将日志输出到文件
    -v <format> 设置日志输出格式控制输出字段，格式如下，默认是brief格式
    	brief--显示优先级/标记和原始进程PID
        process--仅显示进程PID
        tag--仅显示优先级/标记
        thread--仅显示进程：线程和优先级/标记
        raw--显示原始的日志信息，调用时间，优先级/标记，PID
        time--显示日期，调用时间，优先级标记/pid
        long--显示所有的元数据并用空行分割消息内容
    -b <buffer> 加载一个可使用的日志缓冲区供查看，默认是main
    	radio--查看包含在无线/电话相关的缓冲区信息
        events--查看事件相关的消息
        main--查看主缓冲区
        
    adb logcat -f c:/log.txt
    adb logcat -v thread // 使用thread输出格式
    adblogcat -b radio
#### adb shell命令

/system/bin下或者sdk sources/android-20/com/android/commands

##### [pm] 

###### adb shell pm list package [-s|-3|-f|-i] FILTER
	
    adb shell pm list package -i
###### adb shell pm path PATH

	adb shell pm path com.tencent.mobileqq
###### adb shell pm list instrumentation

###### adb shell pm dump

	adb shell pm dump com.tencent.mobileqq
###### adb shell pm install/uninstall

###### adb shell pm clear

###### adb shell pm set-install-location/get-install-location

##### [am]

###### adb shell am start [-n|-S|-W|-a] PACKAGENAME

	adb shell am start -n com.android.camera/.Camera
###### adb shell am force-stop [PACKAGENAME]

###### adb shell am startservice [CLASSNAME]

###### adb shell am broadcast [CLASSNAME]

###### adb shell am monitor

###### adb shell am instrument

##### [input]

###### adb shell input text [TEXT]

###### adb shell input keyevent [KEYCODE]

###### adb shell input tap [X Y]

###### adb shell input swipe [X0 Y0 X1 Y1]

##### [screencap|screenrecord]

###### adb shell screencap -p [PATH]

###### adb shell screenrecord [PATH]

##### [UI automator]

###### adb shell uiautomator [runtest|dump]

##### [ime]

###### adb shell ime list -s

###### adb shell ime set [INPUTMETHOD]

##### [wm]

###### adb shell wm size

##### [monkey]

[Android Monkey的用法](http://xuxu1988.com/2015/05/14/2015-05-02-Monkey/)

##### [settings]

修改系统设置
[探究下 Android4.2 中新增的 settings 命令](http://testerhome.com/topics/1993)

##### [dumpsys]

[参考1](http://blog.csdn.net/zhoumushui/article/details/49175097)

[参考2](https://www.cnblogs.com/JianXu/p/5376642.html)

命令|功能
-|-
package|包查询
activity|所有activity信息
connectivity|网络连接
netpolicy|网络策略
netstats|网络状态
wifi|wifi信息
network_manager|网络管理
account|账号信息
alarm|闹钟信息
meminfo|内存信息
cpuinfo|cpu使用情况
gfxinfo|帧率信息
display|显示
power|电源
batterystats|电池状态
battery|电池
batteryinfo $package_name|电量信息及CPU使用时长
diskstats|	磁盘相关信息
usagestats|每个界面的启动时间
statusbar|状态栏
alarm|闹钟
location|位置
window|窗口


##### [log]

	adb shell log [-p PREVIOUS] [-t TAG] [MESSAGE]

##### [getprop]

查看Android设备的参数信息

	adb shell getprop [key]
#### linux命令

	// 常用命令
    cat,cd,chmod,cp,data,df,du,grep,kill,ln,ls,lsof,netstat,ping,ps,rm,rmdir,top,touch,>,>>,|