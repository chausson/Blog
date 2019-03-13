---
title: CHWebView搞定JS与OC交互
categories: 组件 #分类
date: 2016-08-23 13:31:45
#tags: [博客] #WebView
tags:
---

之前写过一篇文章分析[UIWebView与WKWebView](http://chausson.github.io/2016/08/09/UIWebView%E4%B8%8EWKWebView/),在这基础之实现继承自UIView的控件
CHWebView ,将它作为一个Container包装UIWebView和WKWebView

Github地址：https://github.com/chausson/CHWebView
（给开源的一个Star,谢谢）

# 实现的功能
* 实现JS与OC的调用
* 自带默认进度条属性和协议
* 将UIWebView和WKWebView统一API

# 如何调用OC的方法

1.Object-C需要使用CHWebView并且注册函数名称

2.注册接收对象(使用CHWebViewController的话默认已经注册)

3.实现注册的函数

类似这样
``` obj-c
- (NSArray<NSString *> *)registerJavascriptName{
    return @[@"fetchMessage",@"show"];
}
- (NSObject *)registerJavaScriptHandler{
    return self;
}
- (void)fetchMessage:(NSDictionary *)dic{
   
}
- (void)show{
  
}
```

4.JS如何调用?

当OC这边都准备完毕,通过一行代码即可实现调用OC的方法

OC将方法名获取后通过RunTime的机制发送消息给注册的对象

html这边可以直接使用NativeBridge对象了,NativeBridge({name},{param})

name '表示OC的函数名称'

param '表示需要传入的参数，支持Array,int,object,json'

像这样
``` javascript
   function nativeFounction() {
       var obj = { 'message' : 'Hello, JS!', 'numbers' : [ 1, 2, 3 ] };
       window.NativeBridge('fetchMessage',obj)
   }
    function showUIFuction(){
       window.NativeBridge('show')
    }
```

CHWebView中使用的是WKWebView基于webkit的支持，WKWebView在加载这个Web的时候，默认会注册一个webkit.messageHandlers的对象，同时我在加载完成后注入了一个NativeBridge的对象，目的是为了webkit.messageHandlers和JSContent注入对象API的统一。当然也可以自己手动在加载web的时候导入这段JS
``` javascript
  window.NativeBridge = function(name,message){
    var iosDevice = !!navigator.userAgent.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/);
    if(iosDevice){
       // Apple
       if(this.hasOwnProperty(name)){
            eval("window."+name+"(message)")
       }else{
            if(message == null){
                message = ''
           }
            eval("webkit.messageHandlers."+name+".postMessage(message)")
       }
    }
}
```


# OC调用JS方法
WebView本身就支持调用js的方法，所以很方便，只需要实现CHWebView中的invokeJavaScript:把JS执行的方法传入就可以。

# 进度条属性
CHWebView中有一个协议是更新当前web加载进度的，UIWebView中是在web加载的时候添加一个监听器去监听每次的请求数量作为一个进度加载，WKWebView则是对系统默认的进度条属性进行监听。
``` obj-c
	- (void)webView:(CHWebView *)webVie  updateProgress:(NSProgress *)progress;
```

# CHWebView设计图

<img src="/img/CHWebView设计图.png"  title="CHWebView设计图">

# Demo

<img src="/gif/WebView.gif"  title="CHWebView">

