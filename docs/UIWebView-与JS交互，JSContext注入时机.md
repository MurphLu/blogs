我们做WbeView与js交互，很多时候是使用JavaScriptCore来进行操作，但是使用JavaScriptCore，有时候方法注入时机不对，可能会导致无法正确调用。
比如在html页面刚刚加载的时候js需要调用OC的某个方法，有时候将方法注册写在`-(void)webViewDidStartLoad:(UIWebView *)webView`中可能会无法正常调用到。
解决办法：
创建一个NSObject的Category 命名为"NSObject+JSAdditional"
添加方法
```Objective-C
- (void)webView:(id)unuse didCreateJavaScriptContext:(JSContext *)ctx forFrame:(id)frame {
    [[NSNotificationCenter defaultCenter] postNotificationName:kWebviewCreateContext object:ctx];
}
```

在PrefixHeader中添加引用“NSObject+JSAdditional.h”
WebView所在的ViewController中添加监听kWebviewCreateContext的方法
最后在监听方法中添加要注入的方法，这样无论在WebView加载html的任何时机js都可以正常调用OC方法。

```Objective-C
-(void)addContextFuncs:(NSNotification *)noti{
    JSContext *context = noti.object;
    self.context = context;
    [self setWebViewContext]; //设置需要注入的WebViewContext
}
```

整个过程就是在UIWebView创建了JSContext之后会发出一个通知，接收到通知后立即注入OC方法，而UIWebView的代理方法最早也是要在网页开始加载的时候才去注入，可能注入时机就稍稍晚一些

有哪里表达不准确希望大神指正
