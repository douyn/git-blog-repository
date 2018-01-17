---
title: Hencode学习笔记-1.3
date: 2018年1月17日15:29:14
---
# 文字的绘制

## 1. Canvas绘制文字
canvas绘制文字有三种方式：1. drawText() 2. drawTextRun() 3. drawTextOnPath()
#### 1.1 drawText(String text, float x,  float y, Paint paint)
> 这个坐标可以认为是在文字左下角。是比较接近的位置，但是如果要精确还是以下边所说的基线为准。

#### 1.2 drawTextRun() 
API 23新加入的方法，比drawText有两个新的参数上下文和文字方向， 主要用于文字结构比较特殊的文字的绘制。

#### 1.3 drawTextOnPath()
沿着一条线来绘制文字

	mPaint.setStyle(Paint.Style.STROKE);
    mPaint.setStrokeWidth(2);
    mPaint.setColor(Color.BLUE);

    Path path = new Path();
    path.moveTo(100, 100);
    path.lineTo(200, 200);
    path.rLineTo(300, 100);
    path.rLineTo(400, 200);
    canvas.drawPath(path, mPaint);

    canvas.drawTextOnPath("Hello HenCode!", path, 0, 0, mPaint);

方法为drawTextOnPath(String text, Path path, float hOffset, float vOffset, Paint paint), hOffset和vOffset分别是文字相对于path水平方向和垂直方向的偏移量

#### 1.4 StaticLayout
canvas.drawText()只能绘制单行的文字，而不能换行。

* 不能在view边缘自动换行
* 不能使用\n换行

如果要绘制多行文字，必须把文字切断后使用drawText或者一一使用StaticLayout。

他是android.text.Layout的子类，单纯用来绘制文字的。

它可以设置宽度上限来使文字自动换行，或者使用\n使文字换行。

	String text1 = "Lorem Ipsum is simply dummy text of the printing and typesetting industry.";  
	StaticLayout staticLayout1 = new StaticLayout(text1, paint, 600,  
        Layout.Alignment.ALIGN_NORMAL, 1, 0, true);
	String text2 = "a\nbc\ndefghi\njklm\nnopqrst\nuvwx\nyz";  
	StaticLayout staticLayout2 = new StaticLayout(text2, paint, 600,  
        Layout.Alignment.ALIGN_NORMAL, 1, 0, true);

	...

	canvas.save();  
	canvas.translate(50, 100);  
	staticLayout1.draw(canvas);  
	canvas.translate(0, 200);  
	staticLayout2.draw(canvas);  
	canvas.restore(); 
## 2. Paint对文字绘制的辅助
#### 2.1 设置显示效果
setTextSize() // 设置文字大小

setTextScaleX() // 设置文字缩放

setTextAlign() // 设置对齐方式，三种LEFT,RIGHT,CENTER,默认LEFT

setHinting() // 设置字体微调

现在的 Android 设备大多数都是是用的矢量字体。矢量字体的原理是对每个字体给出一个字形的矢量描述，然后使用这一个矢量来对所有的尺寸的字体来生成对应的字形。由于不必为所有字号都设计它们的字体形状，所以在字号较大的时候，矢量字体也能够保持字体的圆润，这是矢量字体的优势。不过当文字的尺寸过小（比如高度小于 16 像素），有些文字会由于失去过多细节而变得不太好看。 hinting 技术就是为了解决这种问题的：通过向字体中加入 hinting 信息，让矢量字体在尺寸过小的时候得到针对性的修正，从而提高显示效果。效果图盗一张维基百科的：

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig65wwv1yj20ki0bywje.jpg)

**setTextSkewX() // 设置文字倾斜**

setTextLocation() // 设置语言区域

**setLetterPadding() // 设置字符间距**

**setFontFeatureSettings() // 设置字体样式**
	
	paint.setFontFeatureSettings("scmp"); // 设置为small cap
	canvas.drawText("Hello Hencode", 100, 100, paint);

这个东西不懂，以后[补充一下](https://www.w3.org/TR/css-fonts-3/#font-feature-settings-prop)

**setTypeFace() // 设置字体**
	
	paint.setTypeface(Typeface.DEFAULT);

	paint.setTypeface(Typeface.SERIF);
	
	paint.setTypeface(Typeface.createFromAssert(getContext().getAssert(), "xx.ttf"))
setUnderlineText() // 是否加下划线
setFakeBoldText() // 设置伪粗体
setStrikeThruText() // 是否加删除线

**setElegantTextHeight(boolean elegant) // 是否开启优雅的高度**

1. 把「大高个」文字的高度恢复为原始高度；
2. 增大每行文字的上下边界，来容纳被加高了的文字。

**setSubpixelText() // 是否开启次像素级抗锯齿，就是增强抗锯齿效果。**

**setLinearText() // 设置liear text**

#### 2.2 测量文字尺寸

##### 2.2.1 getFontSpacing() // 获得推荐的行距,即是相邻的baseline之间的距离，如下图所示

	canvas.drawText(texts[0], 100, 150, paint);  
	canvas.drawText(texts[1], 100, 150 + paint.getFontSpacing, paint);  
	canvas.drawText(texts[2], 100, 150 + paint.getFontSpacing * 2, paint);  

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig66d27efj20ev06j401.jpg)

##### 2.2.2 getFontMatrics() // 获得FontMatrix， FontMatrix是一个相对专业的工具类，提供了几个文字排印方面的数值：ascent, descent, top,bottom, leading;

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig66iud3gj20ik0bn41l.jpg)

这个不知道什么时候用到
##### 2.2.3 getTextBounds(String text, int start, int end, Rect bounds) // 获得文字的显示范围
text是文字，start,end是指待测的文本的开始位置和结束位置，rect是测量完要赋值的矩形。

	paint.setStyle(Paint.Style.FILL);  
	canvas.drawText(text, offsetX, offsetY, paint);
	
	paint.getTextBounds(text, 0, text.length(), bounds);  
	bounds.left += offsetX;  
	bounds.top += offsetY;  
	bounds.right += offsetX;  
	bounds.bottom += offsetY;  
	paint.setStyle(Paint.Style.STROKE);  
	canvas.drawRect(bounds, paint);  
![](https://ws3.sinaimg.cn/large/52eb2279ly1fig66pdyg4j20ct02tmxf.jpg)

#### 2.2.4 measureText(String text) //  测量文字宽度 

	canvas.drawText(text, offsetX, offsetY, paint);  
	float textWidth = paint.measureText(text);  
	canvas.drawLine(offsetX, offsetY, offsetX + textWidth, offsetY, paint);  

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig671on56j20or04a0te.jpg)

> 注意： getTextBounds()和measureText()的区别：
> 
> 如果你用代码分别使用 getTextBounds() 和 measureText() 来测量文字的宽度，你会发现 measureText() 测出来的宽度总是比 getTextBounds() 大一点点。这是因为这两个方法其实测量的是两个不一样的东西。
> 
> * getTextBounds()测量的是文字的显示范围，这个矩形恰好能够包裹住文字
> * measureText()测量的是文字绘制时所占的宽度，包括了字符间距等

#### 2.2.5 getTextWidths(String text, float[] widths) // 获取每个字符的宽度，并传入width数组中

#### 2.2.6 breakText() // 截取某长度的字符串
google api是这样写的
Measure the text, stopping early if the measured width exceeds maxWidth.意思是测量文本，如果宽度超过给定的宽度就停止测量。其效果就是如果你的字符串的长度大于breakText()中指定的最大长度，drawtext的时候就只显示这个被截取的定长的文字。返回值为截取的字符的个数。

	String text = "Hencode is very very very ... nice!";
    mPaint.setTextSize(64);
    int i = mPaint.breakText(text.toCharArray(), 0, 17, 600, new float[]{});
    canvas.drawText(text.toCharArray(), 0, i, 100, 100, mPaint);

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig67950cnj21080m4grf.jpg)

#### 2.2.7 光标相关
对于像EditText一样的场景，就会需要绘制光标，在API 23开始引入两个新方法来计算光标。(这跟光标没有毛线关系貌似，主要是测量字符的宽度)
##### 2.2.7.1 getRunAdvance(String text, int start, int end, int contextStart, int contextEnd, boolean isRtl, int offset) // 


params | des
-- | --
text | CharSequence: the text to measure. Cannot be null.
start | int: the index of the start of the range to measure
end | int: the index + 1 of the end of the range to measure
contextStart | int: the index of the start of the shaping context
contextEnd | int: the index + 1 of the end of the shaping context
isRtl | boolean: whether the run is in RTL direction
offset | int: index of caret position

return : width measurement between start and offset

即是说 text指的是原文本，start,end指的是要测量的开始和结束的位置，contextStart，contextEnd不知道什么意思， isrtl是否反转字符串，offset指要插入的位置。

返回的是start和offset之间的距离。

	mPaint.setTextSize(36);
    // 包含特殊符号的绘制（如 emoji 表情）
    String text = "Hello HenCoder \uD83C\uDDE8\uD83C\uDDF3"; // "Hello HenCoder 🇨🇳"
    int start = 0;
    int end = text.length();
    int contextStart = 0;
    int contextEnd = text.length();
    boolean isRtl = false;
    int offset = text.length();
    float advance = mPaint.getRunAdvance(text, start, end, contextStart, contextEnd, isRtl, offset - 4);
    canvas.drawText(text, 100, 100, mPaint);
    canvas.drawLine(100 + advance, 60, 100 + advance, 100, mPaint);

![](https://ws3.sinaimg.cn/large/52eb2279ly1fig67pon53j215s0owdru.jpg)
##### 2.2.7.2 getOffsetForAdvance(CharSequence text, int start, int end, int contextStart, int contextEnd, boolean isRtl, float advance) // 给出一个位置的像素值，计算出文字中最接近这个位置的字符偏移量

##### 2.2.8 hasGlyph(String string) // 检查指定的字符串中是否是一个单独的字形。

![](https://ws1.sinaimg.cn/large/006tNc79ly1flgaf31rskj31120damyn.jpg)

## 练习

## 小结