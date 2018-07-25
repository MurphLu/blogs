最近项目需要，研究了一下开屏广告
大概实现思路就是在  ```appdelegte```  中的  ```didFinishLaunchingWithOptions``` 方法中给 ```keyWindow``` 添加一个遮罩```View```，给```View```添加一个计时器，到指定时间将其从```keyWindow```上移除，如果需要其他效果如：点击广告图片跳转到广告详情页面之类的，可以在点击广告图片时发送一个通知，让其他```ViewController```进行跳转之类的（这部分就不用写了，我们主要讨论的是如何添加一个开屏广告）
闲话少絮，下面是代码，其实很简单：

```
- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions{
	//设置self.Window为keyWindow并显示
	[self.window makeKeyAndVisible] // 第一步
	//初始化UIWindow
	self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
	//初始化根控制器
	LJYRootViewController *rootVC = [[LJYRootViewController alloc] init];
	//设置self.window的根控制器为rootVC
	self.window.rootViewController = rootVC; //第二步 
	//初始化开屏广告View
	LJYADView *adView = [[LJYADView alloc] initFrame:[[UIScreen mainScreen] bounds]];
	//将广告View添加到self.Window上。
    //这一步要写在指定keyWindow之后，也就是第二步之后，因为后设置rootVC的话rootVC会覆盖在adView之上
    //而且第一步设置keyWindow要写在最前，写在后边也会有问题
	[self.window addSubview:adView]; //第三步
}
```

这样写完之后可能在进入开屏广告的时候会闪一下```rootVC```界面再显示开屏广告，解决这个问题可以在开屏广告```View```上加一个全屏```imageView```，```imageView```的图片设置为启动图片就不会有这个问题了，好了，大概就是这样吧
同样的方法也可以实现app新版本特性页展示，这样有一个好处就是广告页或者新版本特性页展示完之后不用再等app首页加载，如果通过切换根控制器来实现这个功能的话，还要有等待首页加载的过程，所以在```keyWindow```上加```View```实现体验要更好一些
