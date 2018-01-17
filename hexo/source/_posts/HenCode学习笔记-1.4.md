---
title: [HenCode学习笔记]-1.4 Canvas对绘制的辅助
date: 2018年1月17日15:28:20
---
# Canvas对绘制的辅助
本文主要两个知识点： 范围剪切和几何变换
## 1. 范围剪切
范围裁剪有2个API，分别是clipRect()和clipPath()

#### 1.1 clipRect() // 裁剪一个矩形区域
	
	canvas.save();  
	canvas.clipRect(left, top, right, bottom);  
	canvas.drawBitmap(bitmap, x, y, paint);  
	canvas.restore();   

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig5rzjhx8j21cq0r2aig.jpg)
#### 1.2 clipPath() // 裁剪出path指定的形状

	canvas.save();  
	canvas.clipPath(path1);  
	canvas.drawBitmap(bitmap, point1.x, point1.y, paint);  
	canvas.restore();

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig5sems3tj21920ma7bk.jpg)

## 2. 几何变换
几何变换大致分为3类:

* 直接通过canvas提供的方法实现简单的二维变换
* 通过Matrix来做复杂的二维变换
* 通过camera来做三维变换

#### 2.1 使用canvas做简单二维变换
canvas.translate(float dx, float dy); // 平移(x，y平移距离)

canvas.rotate(float degree, float px, float py) // 旋转(旋转角度，轴心位子)

canvas.scale(float scaleX, float scaleY, float px, float py) // 缩放(x轴缩放比例，y轴缩放比例，轴心位置) 

canvas.skew(float sx, float sy) // 倾斜(x轴倾斜系数， y轴倾斜系数)

sx,sy表示倾斜度数的tan值。

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig5ug5zo7j20pu0d6dl7.jpg)

#### 2.2 使用Matrix做二维变换

##### 2.2.1 使用Matrix做简单的二维变换

Matrix做常见变换的方式：

1. 创建Matrix对象
2. 调用Matrix的 pre/post Translate/Rotate/Scale/
3. 方法来设置几何变换
3. canvas调用setMatrix(matrix)或者contact(matrix)方法来把几何变换应用到canvas。

代码流程如下

	Matrix matrix = new Matrix();
	...
	matrix.reset();  
	matrix.postTranslate();  
	matrix.postRotate();
	
	canvas.save();  
	canvas.concat(matrix);  
	canvas.drawBitmap(bitmap, x, y, paint);  
	canvas.restore();  

##### 2.2.2 使用Matrix做自定义变幻
Matrix使用自定义变换使用的是setPolyToPoly()方法。

poly的意思是多。setPolyToPoly()的作用就是通过多点映射的方式来直接设置变换。

	Matrix matrix = new Matrix();  
	float pointsSrc = {left, top, right, top, left, bottom, right, bottom};  
	float pointsDst = {left - 10, top + 50, right + 120, top - 90, left + 20, bottom + 30, right + 20, bottom + 60};
	
	...
	
	matrix.reset();  
	matrix.setPolyToPoly(pointsSrc, 0, pointsDst, 0, 4);
	
	canvas.save();  
	canvas.concat(matrix);  
	canvas.drawBitmap(bitmap, x, y, paint);  
	canvas.restore(); 

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig5uw4t2jj20q60fcag4.jpg)

#### 2.3 使用camera做三维变换
camera的三维变换有3种，旋转，平移，移动相机。

##### 2.3.1 旋转
Camera.rotate*() 一共有四个方法： rotateX(deg) rotateY(deg) rotateZ(deg) rotate(x, y, z)

	canvas.save();

	camera.save(); //保存Camera的状态
	camera.rotateX(30); // 旋转 Camera 的三维空间  
	camera.applyToCanvas(canvas); // 把旋转投影到 Canvas
	camera.restore(); // 恢复Camera的状态

	canvas.drawBitmap(bitmap, point1.x, point1.y, paint);  
	canvas.restore();  

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig5vam5fij20rq0g0agl.jpg)

这样旋转之后的图形并不是对称，需要配合canvas.translate()

	canvas.save();

	camera.save(); // 保存 Camera 的状态  
	camera.rotateX(30); // 旋转 Camera 的三维空间  
	canvas.translate(centerX, centerY); // 旋转之后把投影移动回来  
	camera.applyToCanvas(canvas); // 把旋转投影到 Canvas  
	canvas.translate(-centerX, -centerY); // 旋转之前把绘制内容移动到轴心（原点）  
	camera.restore(); // 恢复 Camera 的状态
	
	canvas.drawBitmap(bitmap, point1.x, point1.y, paint);  
	canvas.restore();  

> Canvas 的几何变换顺序是反的，所以要把移动到中心的代码写在下面，把从中心移动回来的代码写在上面。

##### 2.3.2 平移 camera.transilate(float x, float y, float z)

##### 2.3.3 移动相机 camera.setLocation(x, y, z)
设置相机的位置

