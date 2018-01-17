---
title: [HenCode学习笔记]-1.2 Paint详解
date: 2018年1月17日15:28:20
---
# [HenCode学习笔记]-1.2 Paint详解

## 概述
Paint类的API大致分为4种

* 颜色
* 效果
* drawText()相关
* 初始化

## 1. 颜色
canvas绘制的内容，有三层对颜色的处理
![](https://ws3.sinaimg.cn/large/52eb2279ly1fig6dcywn2j20j909yabu.jpg)
#### 1.1 基本颜色
设置基本像素颜色的方式

* canvas.drawColor()/canvas.drawRGB()/canvas.drawARGB() // color参数
* canvas.drawBitmap() 通过bitmap设置颜色 // bitmap参数
* 通过paint设置颜色 // paint参数

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig6gxcusnj20iw04xmzr.jpg)

Paint设置颜色的方式有两种：
一种是直接调用Paint.setColor()/paint.setARGB()设置颜色，另一种是通过shader来指定颜色。

##### Shader
着色器Shader,它有几个子类:

* LinearGradient // 线性渐变
* RadialGradient // 辐射渐变
* SweepGradient // 扫描渐变
* BitmapShader // Bitmap着色器
* ComposeShader // 混合着色器

## 2. 效果

## 3. drawText()相关

## 4. 初始化