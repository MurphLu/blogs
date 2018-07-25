哇擦擦，居然有人喜欢用iPhone的粗体模式，设置方式为“设置-通用-辅助功能-Bold Text”，看起来好丑，不过还是有人会用到这个模式的
然而，UIButton在这个模式下同时设置title和image会重叠啊，不知道是不是哪里的bug，当然你的button设置的比这两个东西加起来宽好多还能正常显示，但是有时候就不会这么好了，例如你使用了UIButton的sizeToFit。。显示效果是这个样子的：


![Snip20161029_2.png](http://upload-images.jianshu.io/upload_images/1648999-cdc2599c7a626deb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

某些情况下你有不能直接给button的image设置一个很大的imageInsets，因为好多人都是正常模式下的啊，你一改正常模式下看起来就怪怪的了，这时候就得用代码来搞了啊，可以针对于粗体模式下的button修改一个比较大的imageInsets 来让它看起来正常一些，记得当时QA提这个bug的时候很是头大，不知道从哪里下手啊，然后当时貌似是通过打断点来看这两种模式下字体有什么不同，然后再去针对性的搞得，代码大概是这个样子  

```
//粗体模式下的字体是这个：.SFUIText-Semibold，正常的忘记是啥了，不过判断个这个就可以找到了，然后再设置btn的imageEdgeInsets就得以正常显示了
if ([[UIFont systemFontOfSize:12].fontName isEqualToString:@".SFUIText-Semibold"]) {
        self.btn_diamond.imageEdgeInsets = UIEdgeInsetsMake(0, -10, 0, 10);
}
```


![Snip20161029_3.png](http://upload-images.jianshu.io/upload_images/1648999-8e1ea0f5117ded91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

调整完之后大概就是酱婶的，insets具体调多大就自己去试了，可能跟图标大小什么的也有关系，但是正常字体下的显示就是没有任何问题啊，下面是正常字体下并没有做任何调整的button

![Snip20161029_4.png](http://upload-images.jianshu.io/upload_images/1648999-d0cf92beea9a21ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
