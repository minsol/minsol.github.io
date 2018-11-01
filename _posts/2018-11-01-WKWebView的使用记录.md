---
layout:     post
title:      "WKWebView的使用记录"
subtitle:   ""
date:       2018-11-01
author:     "minsol"
header-img: "img/post-bg-universe.jpg"
catalog: true
tags:
    - iOS开发记录
---

#### HTML调用iOS消息写法
```objc
///HTML调用iOS消息写法---Client对应创建Wk时候的MessageHandler name
window.webkit.messageHandlers.Client.postMessage({
    method:"",
    callback:""
})
```
<br>
#### WKWebView

> WKWebView：网页的渲染与展示，通过WKWebViewConfiguration可以进行配置。<br>
>WKWebViewConfiguration：这个类专门用来配置WKWebView。<br>
>WKUserContentController：这个类主要用来做native与JavaScript的交互管理。<br>
    >```objc
    >/*! @abstract Adds a script message handler.
    > @param scriptMessageHandler The message handler to add.
    > @param name The name of the message handler.
    > @discussion Adding a scriptMessageHandler adds a function
    > window.webkit.messageHandlers.<name>.postMessage(<messageBody>) for all
    > frames.
    > */
    >- (void)addScriptMessageHandler:(id <WKScriptMessageHandler>)scriptMessageHandler name:(NSString *)name;
    >```
>WKScriptMessageHandler：这个类专门用来处理JavaScript调用native的方法。


```objc
    // WKWebView
    WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
    configuration.preferences = [[WKPreferences alloc] init];
    configuration.preferences.minimumFontSize = 10;
    configuration.preferences.javaScriptEnabled = YES;
    configuration.preferences.javaScriptCanOpenWindowsAutomatically = NO;

    ///GPWKWebViewDelegate遵守<WKScriptMessageHandler>
    GPWKWebViewDelegate *wkDelegate = [[GPWKWebViewDelegate alloc] initWithDelegate:self];
    configuration.userContentController = [[WKUserContentController alloc] init];
    
    [configuration.userContentController addScriptMessageHandler:wkDelegate name:@"Client"];
    WKWebView *webView = [[WKWebView alloc] initWithFrame:CGRectZero configuration:configuration];
    [webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:self.urlString]]];
    webView.navigationDelegate = self;
    webView.UIDelegate = self;
    [self.view addSubview:webView];
    self.wkWebView = webView;

    在代理方法里面就可以接受
    ///JS 交互
    - (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
        if (![message.name isEqualToString:@"Client"] || ![message.body isKindOfClass:[NSDictionary class]]) return;
        NSString *methodNameString = [message.body valueForKeyPath:@"method"];  /**< 方法名 */
        NSString *callBackString = [message.body valueForKeyPath:@"callback"]; /**< 方法 */
    }
    //调用JS的方法
    webView.evaluateJavaScript(String) { (Any?, Error?) in
        ///code
    }
```
<br>

#### GPWKWebViewDelegate代码
```objc
    GPWKWebViewDelegate.h
    #import <Foundation/Foundation.h>
    #import <WebKit/WebKit.h>
    /// 代理
    @interface GPWKWebViewDelegate : NSObject <WKScriptMessageHandler>
    @property (nonatomic, weak) id <WKScriptMessageHandler> delegate;
    - (instancetype)initWithDelegate:(id <WKScriptMessageHandler>)delegate;
    @end
```
<br>

```objc
    GPWKWebViewDelegate.m
    #import "GPWKWebViewDelegate.h"
    @interface GPWKWebViewDelegate ()
    @end

    @implementation GPWKWebViewDelegate
    - (instancetype)initWithDelegate:(id<WKScriptMessageHandler>)delegate {
        if (self = [super init]) {
            _delegate = delegate;
        }
        return self;
    }

    - (void)dealloc {
        NSLog(@"%s", __func__);
    }

    #pragma mark - WKScriptMessageHandler
    - (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
        if ([self.delegate respondsToSelector:@selector(userContentController:didReceiveScriptMessage:)]) {
            [self.delegate userContentController:userContentController didReceiveScriptMessage:message];
        }
    }
    @end
```


#### swift相关知识
```swift
    //加载之前就加载脚本
    var js = "var no = document.getElementById('no');no.value = '"
    js+=cardNo
    js+= "';var mob = document.getElementById('mob');mob.value = '"
    js+=mobNo
    js+= "';"  
    let script = WKUserScript(source: js, injectionTime: .atDocumentEnd, forMainFrameOnly: true)
    let config = WKWebViewConfiguration()
    config.userContentController.addUserScript(script)
    let webView = WKWebView(frame: view.bounds, configuration: config)
```

