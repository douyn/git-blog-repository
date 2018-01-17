---
title: Android面试题及答案
date: 2018年1月17日15:25:57
---
# Android面试题及答案
###### 1. Android中的ANR
概念:
Android中的ANR全称是application not responding。应用程序无响应。应用程序在指定时间内没有响应用户操作,系统就会抛出ANR.在activity中的响应时间是５秒，在broadcase中的响应时间是１０秒，在service中的响应时间是２０s。注意:anr是系统抛出的异常，程序是捕捉不到的。

解决方法:
1. 运行在主线程的任何方法都应尽量做更少的事情。特别是，Activity应该在他的关键生命周期方法里尽量少的去做创建操作。
2. 应用程序应该避免在广播接受者里做耗时或者计算操作。但不再是子线程里做这些任务，如果要做耗时操作，应该启动一个service。

###### 2. Fragment的生命周期
完整的生命周期依次是:
onAttach() --> onCreate() --> onCreateView() --> onActivityCreated() --> onStart() --> onResume() --> onPaulse() --> onStop() --> onDestroyView() --> onDestroy() --> onDetach()

切换到这个Fragment分别执行:
onAttch() --> onCreate() --> onCreateVIew() --> onActivityCreated() --> onStart() --> onResume()

切换到其他Fragment:
onPause() --> onStop() --> onDestroyView()

从其他Fragment切换回来:
onCreateView() --> onActivityCreated() --> onStart() --> onResume()

###### 3. Service的生命周期
service中的４个手动调用的方法:

	startService(); 
    stopService();
    bindService();
    unbindService();

service中5个生命周期方法:

	onCreate();
    onStartCommand();
    onBind();
    onUnbind();
    onDestroy();
    
startService();　会执行生命周期中的onCreate() --> onStartCommand();如果一个服务已经启动，不会再执行onCreate(),只会执行onStartCommand()；
 
stopService(): 会执行onDestroy()方法，如果一个服务已经被绑定,必须先解绑才能销毁。

bindService(): 会执行onCreate() --> onBind()方法。

unBindService(): 会执行onUnbindService() --> onDestroy()方法。

startService方式启动的服务，在调用者退出之后仍然存在，bindService启动的服务，如果调用者退出，服务也退出。

###### 4. Android动画
Android中有３中动画 帧动画，补间动画，属性动画

帧动画

就是一张张图片一次播放.在Android中的定义方式是在xml文件中定义<animation-list></animation-list>标签。

启动可以定义oneshot属性，true表示执行一次,false表示执行多次

补间动画

补件动画分为４种,scale/translate/alpha/rotate
常用用法

		AlphaAnimation alphaAnimation = new AlphaAnimation(1f, 0.5f);
        alphaAnimation.setDuration(100);
        alphaAnimation.setRepeatCount(11);
        alphaAnimation.start();
        ImageView imageView = null;
        imageView.startAnimation(alphaAnimation);
属性动画
对于对象属性的动画。就是通过不断的对值的操作来实现动画效果。
	
    ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(targetView, propertyName, valueFrom, valueTo);
    objectAnimator.setDuration(1000);
    objectAnimator.setInterceptor(interceptor);
    objectAnimator.start();
    
    
    ValueAnimator animator = ValueAnimator.ofFloat(0f, 300f);
    animator.addUpdateListener();
    animator.addListener();
    animator.start();

###### 5. Activity的启动方式
standard: 标准模式。是acitvity默认的启动模式，假如a启动了b，那么b就会运行在a所在的任务栈里，而且每次启动一个新的activity，不管栈里是否存在，都会创建新的实例。而且在非activity类型的context中启动标准模式的activity会报错，因为context没有任务栈，想要启动就必须制定待启动的activity的标志位为Intent.FLAG_ACTIVITY_NEW_TASK,这个时候activity会以singleTask方式启动。

singleTask:　栈内复用模式。
如果任务栈中存在这个activity的实例，就不会重新创建，而是去复用这个实例，并将这个实例上的所有activity移除栈。假如activity_a启动了activity_b,又从activity_b跳转回activity_a，此时会先判断activity_a的taskAffinity属性对应的栈中是否存在activity_a,如果没有设置taskAffinity对应的任务栈，就默认是启动他的栈。如果存在taskAffinity对应的任务栈并且，activity指定的栈中存在activity_a，就不会重新创建，而是复用这个activity_a，并将上边所有的activity销毁，如果不存在这个activity_a，就会创建activity_a，如果不存在taskAffinity对应的任务栈，就会创建任务栈，并且创建实例

singleTop:　栈顶复用模式。
如果这个Activity处于栈顶，那么将复用这个栈顶的activity,不会产生新的activity实例。例如activity_a启动activity_b，就会去先判断activity_a所在的栈的栈顶元素是否存在activity_b的实例，如果是，则不会重新去创建activity_b，而是直接引用栈顶实例，如果不是，则要去创建。

singleInstantce:　单实例模式。
只有一个实例，并且单独位于一个任务栈中，这个任务栈中只有一个activity。一旦这个activity被激活，就会复用这个实例。

###### 6. Java引用方式
强引用: 强引用是在程序代码中最普遍存在的,使用方式为Object obj = new Object();或者String str = "strong reference";
即使程序现在内存不足,也不会被回收,只是系统会抛出OOM异常

软引用:
用来描述一些有用但是不是必需的对象.只有在内存不足的时候JVM才会去回收该对象.

弱引用:
也是用来描述非必需的对象.但是只要垃圾回收器扫描到他,他就会被回收.

虚引用:
在任何时候都有可能被垃圾回收.

###### 7. 自定义view的基本流程

1. 编写attr.xml文件定义view的属性
2. 编写layout文件
3. 在布局文件中获取自定义属性
4. 重写onMesure
5. 重写onDraw

###### 8. view的绘制流程

###### 9. view的事件传递机制
基础知识

* 所有的touch事件都被封装称MotionEvent对象,包括触摸的事件位置动作等
* 事件类型分为ACTION_DOWN, ACTION_MOVE, ACTION_UP, ACTION_CANCEL,每个动作都是以ACTION_DOWN开始,以ACTION_UP结束
* 对事件的处理分为3类,包括传递dispatchTouchEvent,拦截onInterceptTouchEvent,消费onTouchEvent/onTouch

传递流程

* 事件都是从dispatchTouchEvent(),只要没有拦截就会向下传递,子view可以通过onTouchEvent对事件进行消费
* 父view可以通过onInterceptTouchEvent对事件进行拦截,阻止其向下传递
* 如果事件从上到下的传递过程中一直没有停止,而且最底层的view没有对事件进行消费,则事件会从下向上传递,父view可以对事件进行消费,如果也没有消费,就会传递到activity的onTouch中进行消费
* 如果子view没有消费ACTION_DOWN,之后的事件不会传递过来
* onTouchListener优先于onTouchEvent对事件进行消费

###### 10. Android IPC机制
Android IPC机制是指Android中多进程环境下出现的进程间通信的处理机制.

多进程会出现的问题

* 静态变量失效.
* 线程同步机制失效
* Application可能被创建多次
* sharepreference的可靠性降低

解决IPC的方法

* 通过intent传递bundle数据
* 通过共享文件
* 使用Messenger
* 使用AIDL
* 使用Socket
* 使用ContentProvider

