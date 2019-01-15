最近项目中想要设置`UILabel`中文字的内边距也就是所谓的`Inset`，在`UILable`中没有发现类似的属性，一顿Google自己写了一个`UILabel`的子类来实现这个属性，当然也可以通过给`UILabel`套一个View的方式来实现，不过这个需求是在写完之后发现的，所以通过子类的方式改起来更好一些。

首先找到了一部分说明需要重写`drawText(in rect: CGRect)`这个方法，代码大概就是这个样子，这个方法就是告诉UILabel要将文字显示在什么位置上，距边框的距离是多少：
```Swift
override func drawText(in rect: CGRect) {
        //其中用到的self.edgeInsets是我们自己定义的一个属性，在初始化UILabel实例的时候对其赋值
        super.drawText(in: UIEdgeInsetsInsetRect(rect, self.edgeInsets))
 }
```

重写了这个方法之后发现在自动布局中并没有什么太大的作用，然后我又发现了下面两个东西，
`var intrinsicContentSize: CGSize`这个是用来得到自动布局中`size`的属性
`sizeThatFits(_ size: CGSize) -> CGSize`这个猜测是`sizeToFit`返回的`size`大小

```Swift
override var intrinsicContentSize: CGSize{
        get{
            var size = super.intrinsicContentSize;
            size.width  += (self.edgeInsets.left + self.edgeInsets.right);
            size.height += (self.edgeInsets.top + self.edgeInsets.bottom);
            return size;
        }
    }

override func sizeThatFits(_ size: CGSize) -> CGSize {
        let superSizeThatFits = super.sizeThatFits(size)
        let width = superSizeThatFits.width + edgeInsets.left + edgeInsets.right
        let height = superSizeThatFits.height + edgeInsets.top + edgeInsets.bottom
        return CGSize(width: width, height: height)
}

```

在实现了这个方法之后发现在文字所占的款度恰好等于`UILabel`的宽度时（设置了`edgeInset`的`left`和`right`且不为0）是不会自动换行的，也就是我们的代码写的还是有问题的，虽然重写了上面两个方法，告诉了`UILabel`要将文字画在哪，最终的大小是多少但是`UILabel`在填充的时候仍然是按照文字的宽度与最终`UILabel`的宽度的比较来判断是不是需要换行的，所以恰好这个临界值是不会换行的，所以就会出现上面的情况，其实我们上面做的工作无非就是在`UILabel`设置完内容之后告诉他你要在哪显示内容，你最终的大小是啥，但是最后得到的大小还是`UILabel`用自己的默认方式计算出来的。我们只是在其计算结果上设置了内边距并体现在了最终显示的结果上。
然后又是一顿Google，最后被我发现了`preferredMaxLayoutWidth`这个属性，这个属性就是来指定`UILabel`计算文字`size`的时候的最大宽度时多少，默认情况下是以`UILabel`的宽度来计算的然后我们在`UILabel`的初始化方法中设置一下它的值，最后计算以及显示就会是正常的了，最起码在我的例子中是正常的。

由于我是通过StoryBoard做的所以之在下面的方法中修改了该值，理论上所有的init方法中都应该设置，当然是要在知道`UILabel`的`frame`之后才能计算
```Swift
required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        self.edgeInsets = UIEdgeInsetsMake(10, 5, 10, 5)
        preferredMaxLayoutWidth = self.frame.width - (self.edgeInsets.left + self.edgeInsets.right)
}
```
