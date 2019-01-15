我们在开发中很多时候会遇到需要显示输入框的文本输入长度，一般就直接通过监听textField的文字改变并做处理就好了，但是有时候往往不注意就会出现小bug，如果只是这样写的话   

```Objective-C
- (void)viewDidLoad {
    [super viewDidLoad];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(textDidChange:) name:UITextFieldTextDidChangeNotification object:nil];
}

-(void)textDidChange:(NSNotification *)noti{
    NSString *textFieldStr = self.textField.text;
//监听文字改变，当超出时截取
    if (self.textField.text.length > 10) {
        self.textField.text= [textFieldStr substringToIndex:10];
    }
}
```

就如下面这样

![输入bug.gif](http://upload-images.jianshu.io/upload_images/1648999-47b89c08cbff14d5.gif?imageMogr2/auto-orient/strip)

之所以会这样是因为我们在输入的时候没有判断中文输入法中正在输入的拼音（也就是选中的部分内容所占的长度）在TextField中占的长度，这样当输入我们所要控制长度一半的拼音字母的时候就会自动截取前五个（假如我们设的最大长度为10）或者（10-前面输入内容）/ 2个字符

解决这个问题可以在textDidChange这个方法中判断当前输入法是什么，并且是否有正在输入的拼音，如果有暂时不做处理，没有的话再进行截取，

修改后的textDidChange方法大概是这样的

```Objective-C
-(void)textDidChange:(NSNotification *)noti{
  //首先判断是否设置了最大长度，如果没有则直接返回
    if (self.maxLength == 0) {
        return;
    }
  //获取TextField
    UITextField *textField = (UITextField *) noti.object;
//获取TextField中的内容
    NSString *str = textField.text;
 // 获取键盘输入模式
    NSString *currentInputMode = [[UITextInputMode currentInputMode] primaryLanguage];
 //简体中文输入，包括简体拼音，健体五笔，简体手写
    if([currentInputMode isEqualToString:@"zh-Hans"]) {
        UITextRange *selectedRange = [textField markedTextRange];
        //获取高亮部分
        UITextPosition *position = [textField positionFromPosition:selectedRange.start offset:0];
        //如果没有高亮选择的字，则对已输入的文字进行字数统计和限制，有高亮的字则不做处理
        if(!position) {
            if(toBeString.length > self.maxLength) {
                textField.text = [str substringToIndex:self.maxLength];
            }
        }
    }
    //中文输入法之外的输入则不需要再加以判断直接截取就ok了
    else{
        if(toBeString.length > self.maxLength) {
            textField.text= [toBeString substringToIndex:self.maxLength];
        }
    }
}
```

然后我们来看一下效果

![输入正常.gif](http://upload-images.jianshu.io/upload_images/1648999-082f6db86633c96e.gif?imageMogr2/auto-orient/strip)


是不是可以爽快的进行输入了，没有任何问题，哈哈哈，ok，就这样解决掉了这个问题
哪里表达不准确欢迎大神指正~~~
