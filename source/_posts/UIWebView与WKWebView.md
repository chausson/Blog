---
title: UIWebView与WKWebView
categories: 小知识 #分类
date: 2016-08-09 11:32:33
tags:
---
# UIWebView
UIWebView是苹果继承于UIView封装的一个加载web内容的类,它可以加载任何远端的web数据展示在你的页面上，你可以像浏览器一样前进后退刷新等操作。不过苹果在iOS8以后推出了WKWebView来加载Web，下面再详细介绍下WKWebView。

UIWebView属于UIKit，封装了WebKit.framework的WebView.

WebView组合管理了WebCore.framework的Page,并提供了各种Clients.

Page管理了Main Frame，Main Frame管理了sub Frame（FrameTree)
[关于详细的UIWebView介绍转自这里](http://blog.csdn.net/hursing)
<img src="/img/webview.png"  title="webview层级">
WebView继承自WAKView,WAKView类似于NSView，可以做较少的改动使得Mac和iOS共用一套。由UIWebDocumentView对WebView进行操作并接收回调事件，当数据发生变化的时候，就会通知UIWebTiledView重新绘制。

UIWebTiledView和WAKWindow这两个类主要负责页面的绘制，包括布局绘图排版，交互等,WAKWindow还会做一些用户操作事件的分派。

UIWebBrowserView主要负责
* form的自动填充
* fixed元素的位置调整
* JavaScript的手势识别
* 键盘弹出时的视图滚动处理，防止遮挡
* 提供接口让UIWebView获取信息
* 为显示PDF时添加页号标签
通过反编译可以获得UIWebViewInternal的具体成员变量
``` obj-c
@interface UIWebViewInternal : NSObject  
{  
    UIScrollView *scroller;  
    UIWebBrowserView *browserView;  
    UICheckeredPatternView *checkeredPatternView;  
    id <UIWebViewDelegate> delegate;  
    unsigned int scalesPageToFit;  
    unsigned int isLoading;  
    unsigned int hasOverriddenOrientationChangeEventHandling;  
    unsigned int drawsCheckeredPattern;  
    unsigned int webSelectionEnabled;  
    unsigned int drawInWebThread;  
    unsigned int inRotation;  
    NSURLRequest *request;  
    int clickedAlertButtonIndex;  
    UIWebViewWebViewDelegate *webViewDelegate;  
    UIWebPDFViewHandler *pdfHandler;  
} 
@end 
```
由此可以看出UIWebViewInternal是接收WebView的事件的载体通过自身把WebView的事件传递给UIWebView.

# WKWebView
通过上面的了解，苹果终于在8.0之后开放了WKWebView应用于iOS和OSX中，它取代了UIWebView和WebView，在两个平台上支持同一套API。
它脱离于UIWebView的设计，将原本的设计拆分成14个类，和3个代理协议，虽然是这样但是了解之后其实用法比较简单，依照职责单一的原则，每个协议做的事情根据功能分类。

## WKWebView相比于UIWebView
* WKWebView的内存远远没有UIWebView的开销大,而且没有缓存
* 拥有高达60FPS滚动刷新率及内置手势
* 支持了更多的HTML5特性
* 高效的app和web信息交换通道
* 允许JavaScript的Nitro库加载并使用,UIWebView中限制了
* WKWebView目前缺少关于页码相关的API
* 提供加载网页进度的属性

## WKWebView的协议
### WKScriptMessageHandler协议
``` javascript
window.webkit.messageHandlers.{NAME}.postMessage()
```
可以把JavaScript对象通过该API自动转换成Objective-C或Swift 对象,Name可以通过addScriptMessageHandler: name:来设置
``` obj-c
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{
}
```
作为唯一响应JavaScript的协议方法，目的是为了与其它的进行分离，在该协议中响应之前注入的MessageHandlers.
可以根据WKScriptMessage知道Js的名称和参数，来区分不同的响应事件
### WKNavigationDelegate协议
提供了追踪主窗口网页加载过程和判断主窗口和子窗口是否进行页面加载新页面的相关方法,相当于UIWebView中webViewDidFinishLoad和webViewDidStartLoad方法,除了有开始加载、加载成功、加载失败的API外，还具有额外的三个代理方法：
``` obj-c
- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(WKNavigation *)navigation;

- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler;

- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler;
```
第一个是服务器redirect时调用
第二个API是根据客户端受到的服务器响应头以及response相关信息来决定是否可以跳转
第三个API是根据WebView对于即将跳转的HTTP请求头信息和相关信息来决定是否跳转
## WKUIDelegate协议
提供用原生控件显示网页的方法回调,例如Alert提示可以自定义用原生的控件来实现
``` obj-c
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler {
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:nil message:message preferredStyle:UIAlertControllerStyleAlert];
    [alert addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        completionHandler();
    }]];
    [self presentViewController:alert animated:YES completion:NULL];
}
```
# 总结
WKWebView相较于UIWebView在整体上有较大的提升，满足OS上面使用同一套控件的功能，同时对整个内存的开销以及滚动刷新率和JS交互做了优化的处理。依据职责单一的原则，拆分成了三个协议去实现WebView的响应，解耦了JS交互和加载进度的响应处理。WKWebView没有做缓存处理,所以对网页需要缓存的加载性能要求没那么高的还是可以考虑UIWebView.
