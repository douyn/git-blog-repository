---
title: Android webview和H5交互
date: 2018-01-12 09:59:19
---
# Android webview和H5交互

## 1. WebView加载页面
webview可以加载本地和网络页面,根据html的文件位置,有不同的写法.
	
     mWebView.loadUrl("www.baidu.com");
     mWebView.loadUrl("file:///android_res/test.html");
     
通常情况下,webview会重新设置webchromeclient,以便在本应用内实现页面跳转.
	
    mWebView.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                mWebView.loadUrl(url);
                return true;
            }
        });
        
另外,要实现本文的与h5交互必须允许使用js接口,在实际开发中,一般都要加上这一句.

	WebSettings settings = mWebView.getSettings();
    settings.setJavaScriptEnabled(true);
## 2. WebView调用js方法
调用js方法有两种情况: 如果调用js的无返回值方法, 可以直接使用load方法

	mWebView.loadUrl("javascript:do()");
    
如果要调用有返回值方法,需要调用evaluatjavascript方法

	mWebView.evaluateJavascript("sum(1,2)", new ValueCallback<String>() {
            @Override
            public void onReceiveValue(String value) {
                Toast.makeText(ViewPagerGalleryDemoActivity.this, "value = " + value, Toast.LENGTH_SHORT).show();
            }
        });
 
 js代码如下:
 
 	<script type="text/javascript">
        function sum(a,b){
        return a+b;
        }
        function do(){
        document.getElementById("p").innerHTML="hello world";
        }
    </script>

## 3. js调用Android方法
需要在Android中定义javascriptinterface接口和声明
1. 首先要添加一个类和方法,并且用javascriptinterface

	public class TestJavaScriptInterface {
        @JavascriptInterface
        private String testJavaScriptInterface () {
            return "hello javascript"
        }
    }
    
2. 打开javascriptinterface给h5的开关

        mWebView.addJavascriptInterface(new TestJavaScriptInterface(), "android");
        
3. js代码如下:


	<script type="text/javascript">
    function s(){
        //调用Java的back()方法
        var result =window.android.testJavaScriptInterface();
        document.getElementById("p").innerHTML=result;
    }
    </script>
    
    