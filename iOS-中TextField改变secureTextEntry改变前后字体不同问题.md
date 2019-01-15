默认状态下```TextField```字体是这个样子的￼
![QQ20160603-0@2x.png](http://upload-images.jianshu.io/upload_images/1648999-235be88c97fa9811.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是当切换到密文状态再切换回来之后，就变成了下面的样子
![QQ20160603-1@2x.png](http://upload-images.jianshu.io/upload_images/1648999-a5eb7eb84117410e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不知道为什么苹果默认在切换```secure```状态是要改变```TextField```的字体，其实在切换完secure状态变为非密文状态的时候默认给```TextField```设置下字体就好可以解决了，大概就是这样

```Objective-C
@interface ViewController ()
@property (weak, nonatomic) IBOutlet UITextField *textField;
@property (weak, nonatomic) IBOutlet UIButton *changeBtn;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.textField.font = [UIFont fontWithName:@"System" size:15];
}

- (IBAction)changeSecure:(id)sender {
    self.textField.secureTextEntry = !self.textField.secureTextEntry;
    self.textField.font = [UIFont fontWithName:@"System" size:15];
}
```
