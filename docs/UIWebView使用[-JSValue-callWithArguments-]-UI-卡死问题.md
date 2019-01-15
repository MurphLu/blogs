最近一直在搞一个套壳的app，作为临时方案，并且为了能够使交互与安卓端统一，用的UIWebView，体验就不要说了，卡出翔。
在使用UIWebView的时候出现了这样一个问题，当使用[ JSValue callWithArguments:]方法时，如果调用的js方法有alert，就会导致UI卡死，alert点击无效，然后一顿google，stackoverflow。最终找到了解决方案。
```Objective-C
//假如你要调用的js方法名称为 “test”
//先获取webView中js上下文
self.context = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
//获取js方法
JSValue *jsFun = self.context[@"test"];
//异步主线程执行js方法
dispatch_async(dispatch_get_main_queue(), ^{
//使用js的window.setTimeout方法执行需要调用的方法
       [jsFun.context[@"setTimeout"] callWithArguments:@[jsFun, @0, args];
 });
```
这里是该问题的具体描述及解决方案，具体原因还不是很理解，等研究过后再加补充，望各路大神指正
https://stackoverflow.com/questions/22876528/calling-jsvalue-callwitharguments-locks-ui-when-alert-is-called
