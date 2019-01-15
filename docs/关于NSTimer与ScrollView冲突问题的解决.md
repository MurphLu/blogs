有这样一个场景，在`UIScrollView`中添加一个`Label`，通过`NSTimer`去给这个`Label`添加一个倒计时功能，如果在将计时器添加到`RunLoop`中的时候使用了`NSDefaultRunLoopMode`，那么当`ScrollView`滚动时`Label`上的倒计时就会停止，但是如果使用了`NSRunLoopCommonModes`就不会出现这样的情况。  
 
之所以会这样，是因为主线程的`NSRunLoop`默认有两个`Mode`，一个是`NSDefaultRunLoopMode`，另一个是`UITrackingRunLoopMode`，当`ScrollView`滚动时会默认切换到`UITrackingRunLoopMode`，加入只是添加到`NSDefaultRunLoopMode`中，那么切换`Mode`时自然就不会再执行`NSTimer`的方法了，而`NSRunLoopCommonModes`这个`Mode`实际上是`NSDefaultRunLoopMode`、`UITrackingRunLoopMode`的集合体，注意看是` NSRunLoopCommonModes`后面多了一个*s*，其实这个`CommonModes`是默认标记了这两个`Mode`，当给`RunLoop`中添加`Timer`时给这两个`Mode`中同时添加了这个`Timer`，所以当`Mode`切换时就不会有问题了，当然可以同时将`Timer`添加到这两个`Mode`中也能起到同样的效果

如有哪里写的不妥欢迎大神们指正~~
