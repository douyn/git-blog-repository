---
title: 【踩坑记录】自定义ViewGroup
date: 2018年1月17日15:32:31
---
# 【踩坑记录】自定义ViewGroup
#### 1.获取自定义viewgroup对象为空
解决方法:

检查自定义viewgroup的构造方法

	public SearchHistoryView(Context context) {
        this(context, null);
    }

    public SearchHistoryView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public SearchHistoryView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
看构造方法调用是否正确，参数是否正确。我出现这个错误是在两个参数的构造方法中把第二个参数写成了null导致这个错误。

原因:

在自定义viewgroup的类中，构造方法中一般会调用两个参数的构造方法，在写构造方法时注意，一般都是最后调用this(arg1, arg2,arg3)这个构造方法，如果出现对象为空的情况，检查构造方法，参数是否正确，是否调用正确。

#### 2.自定义viewgroup，直接在布局中添加子view正常，代码添加view时出现java.lang.ClassCastException: android.view.ViewGroup$LayoutParams cannot be cast to android.view.ViewGroup$MarginLayoutParams
解决方法:

自定义LayoutParams类，

	public class LayoutParams extends MarginLayoutParams {

        public int gravity = -1;

        public LayoutParams(Context c, AttributeSet attrs) {
            super(c, attrs);
        }

        public LayoutParams(int width, int height) {
            super(width, height);
            gravity = Gravity.TOP;
        }

        public LayoutParams(int width, int height, int gravity) {
            super(width, height);
            this.gravity = gravity;
        }

        public LayoutParams(ViewGroup.LayoutParams source) {
            super(source);
        }
    }
并在自定义viewgroup类中重写如下几个构造方法

	@Override
    protected LayoutParams generateDefaultLayoutParams() {
        return new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
    }

    @Override
    protected boolean checkLayoutParams(ViewGroup.LayoutParams p) {
        return super.checkLayoutParams(p) && p instanceof LayoutParams;
    }

    @Override
    protected LayoutParams generateLayoutParams(ViewGroup.LayoutParams p) {
        return new LayoutParams(p);
    }

    @Override
    public LayoutParams generateLayoutParams(AttributeSet attrs) {
        return new LayoutParams(getContext(), attrs);
    }
    
    
原因:

我是按照网上的zhy大神的flowlayout教程写下来的，他是在自定义viewgroup中重写了generatelayoutparams()方法，然后在onmeasure中直接讲layoutparams直接强转成marginlayoutparams，做demo的时候因为是直接在xml中把子view添加到viewgroup控件中，这种做法是没有问题的，但是在写项目的时候因为需要动态添加布局，这时候就出现了以上的问题。建议去看一下layoutparams的知识。

#### 3.子view设置了margin，但是在viewgroup中看到log打印margin还是0
解决方法：

因为我使用的是动态添加子view的方式

	 TextView textView = (TextView) LayoutInflater.from(SearchActivity.this).inflate(R.layout.item_shv_textview, null, false);
                textView.setText(bean);

上边的代码一般都是我写inflat view的代码，3个参数中,arg1代表资源文件，arg2代表父布局，arg3代表是否依赖父布局，一般情况下用到的时候我都是上边的代码，但是在这里用要改为

	TextView textView = (TextView) LayoutInflater.from(SearchActivity.this).inflate(R.layout.item_shv_textview, mParent, false);
                textView.setText(bean);
              
 就是说必须指定他所在的viewgroup，margin属性才有值.
 
原因：