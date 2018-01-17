---
title: View事件传递
date: 2018年1月17日15:30:47
---
# View事件传递机制

#### 基础概念
1. Android中的touch事件都被封装到TouchEvent对象中，包括触摸的时间，位置，动作等
2. Touch事件的类型有ACTION_DOWN,ACTION_MOVE,ACTION_UP,ACTION_CANCEL等。每个事件都是以ACTION_DOWN开始，以ACTION_UP结束。
3. 对事件的处理有三种，传递，拦截，消费，分别是dispatchTouchEvent(), onInterceptTouchEvent(),onTouchEvent()和onTouchListener

#### 传递流程

1. 事件都是从activity.dispatchTouchEvent()开始的，如果没有被拦截就会一直走view的diapathTouchEvent(),从最上层的ViewGroup一直向下传递到最下层的子View,最后子View可以通过onTouch()消费掉事件。如果子View没有消费掉事件，那么事件就会向上传递，这时ViewGroup可以消费事件，如果也没有消费的话，最后会走activity.onTouchEvent();
2. 事件可以被拦截。ViewGroup可以通过onInterceptTouchEvent()阻止其向下传递。
3. 如果view没有对ACTION_DOWN进行消费，则其他事件不会传递过来。
4. onTouchListener优先与onTouchEvent进行消费.

![View不处理事件流程](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tech/touch-event/image/ignorant-view-example.jpg)

![View处理事件流程](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tech/touch-event/image/interested-view-example.jpg)