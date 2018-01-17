---
title: [HenCode学习笔记] View-1.1 绘制基础
date: 2018年1月17日15:27:44
---
# [HenCode学习笔记] View-1.1 绘制基础

绘制基础对于有一定自定义view经验的开发者来说是比较简单的，所以这里的笔记也简单做一下

## 概述
自定义绘制的方法是重写绘制方法，最常用的是onDraw();

绘制的关键是Canvas:

* 绘制类方法: drawXXX(关键参数 Paint)
* 辅助类方法: 裁剪范围和几何变换

可以使用不同的绘制方法来控制遮盖关系

## 自定义绘制的四个级别

1. 熟练使用drawXXX()和paint的常见用法
2. 熟练使用paint
3. 熟练canvas对绘制的辅助-裁剪和几何变换
4. 使用不同的绘制方法来控制绘制顺序

## onDraw()

## canvas.drawXXX和Paint基础
主要包括
* Canvas类下的所有drawXXX(),例如drawCicle(), drawBitmap()
* Paint类中的几个常见方法,例如
	* Paint.setStyle() // 设置绘制模式
	* Paint.setColor() // 设置画笔颜色
	* Paint.setStrockWidth() // 设置线条宽度
	* Paint.setTextSize() // 设置文字大小
	* Paint.setAntiAlias() // 设置是否开启抗锯齿

## canvas.drawColor() // 颜色填充,设置画布颜色
> drawRGB,drawARGB作用一样

## drawCircle // 画圆
? 参数：圆心x轴坐标，圆心Y轴坐标，圆心半径，画笔Paint提供除基本信息外所有风格信息

## drawRect // 画矩形

## drawPoint // 画点
> 画点不指是圆点，还有方块等

> 点的大小可以通过paint.setStrockWidth()来设置

> 点的形状可以通过paint.setStrockCap()来设置,这个方法其实是用来设置笔触风格，就像现实中的钢笔，毛笔，圆珠笔一样，同样在纸上点一下，会出现不同的形状，这个形状就叫笔触，下图中的线冒也是指的笔触，不过下边的就比较形象。笔触有三种, BUTT(无),SQUARE(方形),ROUND(圆形)
![](http://img.blog.csdn.net/20160330083020037?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

> setStrokeJoin 用来设置接合处的形状。如下图所示![](http://img.blog.csdn.net/20160330083742743?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以设置三种参数Join.MITER(锐角), Join.BEVEL(直角), Join.Round(圆角)

## drawOval(left, top, right, bottom, paint) // 画椭圆，左上右下分别对应椭圆的四个边界

## drawLine(startX, startY, endX, endY, paint)
> 由于直线不是封闭图形，所以setStyle对直线没有影响。(这句话不一定对)

## drawRoundRect() // 画圆角矩形

## drawArc(left, top, right, bottom, startAngle, sweepAngle, useCenter, paint) // 画弧形和扇形
> 左上右下用来描述圆弧所在的椭圆的，startAngle起始的角度(x轴的正向，即正右方向为0度的位置，顺时针为正向，逆时针为负向)，sweepAngle是弧形扫过的角度，useCenter表示是否连接到圆心，即是如果连接到圆心表示扇形，不连接到圆心表示弧形。

## drawPath(path, paint) // 画自定义图形
Path就是用来描述图形路径的对象，这个类中的几个常见方法:

Path.arcTo();

Path.lineTo();

Path.addArc();

...

Path可以描述直线，二次曲线，三次曲线，园，椭圆，弧形，矩形，圆角矩形等，并可以将这些图形组合起来形成各种复杂的图形。

## Path有两种方法:
### 第一种： 直接描述路径，又可以分为两种
##### 1. addXXX -- 添加子图形
addCircle(centerX, centerY, radius, direction)

> Path的方向有两种 CW(顺时针)和CCW(逆时针)，当图形为填充并且图形有自相交时，需要根据方向判断填充范围。
##### 2. xxxTo -- 划线（直线或者曲线）
lineTo(x,y) // 从当前位置向目标位置划线。
rLineTo(x,y) // 从相对当前位置向目标位置划线。

	paint.setStyle(Style.STROKE);  
	path.lineTo(100, 100); // 由当前位置 (0, 0) 向 (100, 100) 画一条直线  
	path.rLineTo(100, 0); // 由当前位置 (100, 100) 向正右方 100 像素的位置画一条直线 
 
quadTo(x1, y1, x2, y2) // 是指从当前位置向目标位置画二次贝塞尔曲线。(x1,y1)是控制点的坐标，(x2,y2)是目标点坐标

cubicTo() // 三次贝塞尔曲线，两个控制点

moveTo(x,y)/rMoveTo(x,y) // 移动当前位置或者相对位置的坐标

> arcTo(rectf, startAngle, sweepAngle, forceMoveTo) // arcTo方法没有UseCenter参数，因为path只能画弧线，forceMoveTo为true表示强制移动到弧线起点位置(无痕迹)，为false表示要把当前位置到弧线起点位置连起来。

> close封闭当前图形。即是将当前图形封闭，我看来是当前位置到相对位置连接起来

	paint.setStyle(Style.STROKE);  
	path.moveTo(100, 100);  
	path.lineTo(200, 100);  
	path.lineTo(150, 150);  
	// 子图形未封闭
	
![](https://ws1.sinaimg.cn/large/006tNc79ly1fig7zidfmtj304p0330sl.jpg)

	paint.setStyle(Style.STROKE);  
	path.moveTo(100, 100);  
	path.lineTo(200, 100);  
	path.lineTo(150, 150);  
	path.close(); // 使用 close() 封闭子图形。等价于 path.lineTo(100, 100)  
	
![](https://ws2.sinaimg.cn/large/006tNc79ly1fig7zqj7llj304n03aglh.jpg)

	paint.setStyle(Style.FILL);
  	path.moveTo(100, 100);
  	path.lineTo(200, 100);
  	path.lineTo(150, 150);
  	// 这里只绘制了两条边，但由于 Style 是 FILL ，所以绘制时会自动封口

![](https://ws3.sinaimg.cn/large/006tNc79ly1fig80a2v95j304z035glh.jpg)

### 第二种: 辅助的设计或者计算
Path.setFillType(fillType) // 前面在说 dir 参数的时候提到，这个方法是用来设置图形自相交时的填充算法的：
![](https://ws3.sinaimg.cn/large/006tNc79ly1fig8042q9jj30hb0aodg6.jpg)

FillType四种类型:
* EVEN_ODD
* WINDING(默认)
* INVERSE_EVEN_ODD
* INVERSE_WINDING

![](https://ws1.sinaimg.cn/large/006tNc79ly1fig7zyzjtrj30kx0jugn9.jpg)

###### ENVEN_ODD
即 even-odd rule （奇偶原则）：对于平面中的任意一点，向任意方向射出一条射线，这条射线和图形相交的次数（相交才算，相切不算哦）如果是奇数，则这个点被认为在图形内部，是要被涂色的区域；如果是偶数，则这个点被认为在图形外部，是不被涂色的区域。还以左右相交的双圆为例：

![](https://ws3.sinaimg.cn/large/006tNc79ly1fig811a7tnj30jj06h74p.jpg)

###### WINDING
即 non-zero winding rule （非零环绕数原则）：首先，它需要你图形中的所有线条都是有绘制方向的：

![](https://ws4.sinaimg.cn/large/006tNc79ly1fig81dw60ej307b0bm3yq.jpg)

然后，同样是从平面中的点向任意方向射出一条射线，但计算规则不一样：以 0 为初始值，对于射线和图形的所有交点，遇到每个顺时针的交点（图形从射线的左边向右穿过）把结果加 1，遇到每个逆时针的交点（图形从射线的右边向左穿过）把结果减 1，最终把所有的交点都算上，得到的结果如果不是 0，则认为这个点在图形内部，是要被涂色的区域；如果是 0，则认为这个点在图形外部，是不被涂色的区域。
![](https://ws3.sinaimg.cn/large/006tNc79ly1fig81scem2j30k90digmo.jpg)

## drawBitmap(bitmap, left, top)

## drawText(text, x, y, paint) // x,y可以认为是绘制起点的坐标

## 练习