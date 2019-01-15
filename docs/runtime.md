最近在研究runtime，所以就简单记录下我的一些收获吧算是

##利用runtime通过分类添加类属性

`iOS`类扩展：`category`可以扩展类的方法，但是不能扩展类的属性，当我们想要给一个已经封装好的类添加属性，但是这个属性只是特定情况才会用到的，这时我们就可以通过`runtime`来通过分类给现有类添加属性了。
首先使用runtime我们需要引入`<objc/runtime.h>`这个头文件然后代码如下  

``` Objective-C
//Person.h

@interface Person (name)

@property (nonatomic,copy)NSString *name;

@end

@interface Person : NSObject

-(void)run:(NSString *)string;

@end

//========================================
//Person.m

@implementation Person (name);
static const char *key = "name";

//通过objc_getAssociatedObject方法，给类属性添加get方法
-(NSString *)name{
    /*
        参数一：对象
        参数二：属性名
    */
    return objc_getAssociatedObject(self, key);
}

-(void)setName:(NSString *)name{
    /*
      参数一：要添加属性的对象
      参数二：属性名
      参数三：属性值
      参数四：属性修饰符
    */
    objc_setAssociatedObject(self, key, name, OBJC_ASSOCIATION_COPY_NONATOMIC);
}
@end

static const char *key = "name";

@implementation Person
@end  
//之后再调用p.name = @"123",或者p.name,就能正常运行了

```

##iOS消息转发机制
`iOS`没有方法调用，作为动态语言它调用方法其实就是给对象发消息，我们可以在它发消息的过程中做很多操作，即使有些方法没有实现，我们也可以通过它消息发送的过程来进行一些处理，来保证程序不会crush。
首先我们来看一下`iOS`消息发送的整个过程
>首先Person对象去调用一个方法的时候，假如这个方法名是`run`，参数为`NSString`
>`-(void)run:(NSString *)run`
>`Person *p = [Person new]; [p run]`

>实际上编译完成之后所执行的方法是`objc_msgSend(p,@selector(run:))`
>
>执行这句代码的时候首先回到该对象的方法列表中找这个方法的实现，如果找到则直接执行
>
>如果没有实现该方法，就会执行它的类方法`+(BOOL)resolveInstanceMethod:(SEL)sel`
>我们可以在这个方法中去给对象添加方法，并且返回`YES`，这样就会执行我们添加的方法，如果并没有做处理，则调用`super`的`resolveInstanceMethod`，如果最终依然没有查找到，则返回`NO`
>
>此时就会默认执行`-(id)forwardingTargetForSelector:(SEL)aSelector`来进行消息转发，我们可以在这里将该方法转发给其他已经实现了这个方法的对象，交由它们去处理，如果在这里并没有进行处理并返回了 `nil`,那我们还有第三次机会来处理这个消息
>
>在进行第三次处理之前首先要实现一个方法，否则上一步返回nil了就会直接crush掉
>`-(NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`这个方法字面上来理解就是返回一个方法签名，当然这个方法签名需要从指定对象中获取，这个对象也就是作为在第三次机会中处理该消息的对象
>
>在实现了上述方法之后我们就可以使用`-(void)forwardInvocation:(NSInvocation *)anInvocation`，方法来进行消息转发的第三次也就是最后一次处理，如果在这里再不进行相应的处理，那就会直接调用`-(void)doesNotRecognizeSelector:(SEL)aSelector`方法，然后crush掉了，当然你不想程序crush的话也可以重写这个方法 来提供log日志等操作  


说了这么多我自己都不知道自己在说什么了，下面我们来看代码，希望能通过代码说明白。。。  

```Objective-C
//首先由Person类 Person.h, 声明方法run
@interface Person : NSObject

-(void)run:(NSString *)string;

@end


//Person类的实现
//  Person.m
@implementation Person

/*
//实现了run方法的话，如果在其他地方调用run方法，则直接执行该方法实现，不会再往下走任何方法
-(void)run:(NSString *)string{
    NSLog(@"========");
}
*/

//假如没有实现run方法，那么给person对象发送消息的时候就会首先来到这个方法，
//我们可以在这个方法中给person对象添加run方法并返回YES，也就是注释部分可以通过
//class_addMethod把我们下面实现的“aaa”方法作为run方法的实现来添加到person对象上，
//这时消息转发到这里就结束了，不会再继续往下走

//假如此时返回了NO,那么就会到下一个方法，来对消息进行转发
+(BOOL)resolveInstanceMethod:(SEL)sel{
//    if (sel == @selector(run:)) {
//        class_addMethod(self, sel, (IMP)aaa, "v@:@");
//        return YES;
//    }
//    return [super resolveInstanceMethod:sel];
    return NO;
}

void aaa(id self, SEL _cmd, id Num){
    NSLog(@"%@的%@方法动态实现了，参数为%@",self, NSStringFromSelector(@selector(_cmd)),Num);
}


//当+(BOOL)resolveInstanceMethod:(SEL)sel返回NO时我们就会来到这里，来进行消息的转发操作
//注释部分内容为消息转发处理操作，我们假设有一个Stu类，声明并实现了run:方法，
//在这里我们可以返回Stu的对象，将run这个消息交由Stu对象处理，这样也会正常执行并调
//用Stu对象的run方法，但是此时入过返回nil，那么我们还有第三次机会来处理这个消息，
//我们往下看
-(id)forwardingTargetForSelector:(SEL)aSelector{
//    if (aSelector == @selector(run:)) {
//        return [Stu new];
//    }
//    return [super forwardingTargetForSelector:aSelector];
    return nil;
}

//-(id)forwardingTargetForSelector:(SEL)aSelector返回为nil时，首先会来到这个方法，
//如果我们不重写这个方法是无法进入到第三次消息转发的方法中对消息进行处理的，
//这里会返回一个方法签名，如果不为nil，则会进入到`-(void)forwardInvocation:(NSInvocation *)anInvocation`
//这个方法中进行第三次消息转发操作，如果是nil的话就说明没有对象可以处理这个消息，
//那么就会crush掉了，实现了这个方法之后我们就可以进入到第三次消息转发处理的方法中了
-(NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector{
    NSMethodSignature *sig;
    sig = [[Stu new] methodSignatureForSelector:@selector(run:)];
    if (sig) {
        return sig;
    }
    return nil;
    
}

//这里我们可以通过`NSInvocation`的`invokeWithTarget`方法来将上面过程中没有处理掉
//的消息进行转发操作，来保证程序的正常运行，如果在这里还是没有处理那就直接到
//`-(void)doesNotRecognizeSelector:(SEL)aSelector`这个方法中了，加入你并不想程序crush，
//也可以重写doesNotRecognizeSelector这个方法，在该方法中进行log日志等的处理
-(void)forwardInvocation:(NSInvocation *)anInvocation{
    if([[Stu new] respondsToSelector:[anInvocation selector]]){
        [anInvocation invokeWithTarget:[Stu new]];
    }else{
        [super forwardInvocation:anInvocation];
    }
    
}

-(void)doesNotRecognizeSelector:(SEL)aSelector{
    NSLog(@"gaobuding");
}


@end


```
iOS的消息转发机制大概就是这样了，我也不知道你们能不能看懂，个人水平有限，暂时只能发挥到这样了。。

##交换方法的实现

我们可以通过method_exchangeImplementations，来交换方法地址达到方法交换的目的
假设有这样一个场景，我们在没有一个类的实现源码的情况下，想改变其中一个方法的实现，除了继承它重写、和借助类别重名方法之外，还有没有其他方法可以搞定呢，当然可以，我们可以用method_exchangeImplementations，在load方法中对类的方法用我们自己的实现来进行交换  

```Objective-C
@implementation Person

+(void)load{
    //获取类默认方法
    Method description = class_getInstanceMethod([Person class], @selector(description));
    //获取我们自己实现的方法
    Method desc = class_getInstanceMethod([Person class], @selector(desc));
    //交换方法
    method_exchangeImplementations(description, desc);
}

-(NSString *)desc{
    //这里使用[self desc]并不会造成循环应用，因为此时desc指向的是默认的description方法，
    //而description方法指向的是desc，在其他地方调用对象的description方法是实际上是走到了我们自定义的desc方法中了，，是不是有点乱。。
    NSLog(@"%@",[self desc]);
    return @"哈哈哈";
}

@end

```
