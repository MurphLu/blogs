这两天研究```class-dump```，本来是想把它放到```user/bin```下用```Terminal```打开的，无奈考不进去，发现Max OS X11之后在内核下引入了Rootless机制，以下路径
```

/System
/bin
/sbin
/usr (except /usr/local)
```
即使是root用户也不能随意修改，这就有点尴尬了。。
于是，本着曲线救国的态度，我在个人用户文件夹下建了个隐藏文件夹，然后将哪些想用```Terminal```打开的东西全都丢在那个文件夹下了，然后将这个路径再加到```path```里，ok，大功告成。
有的小伙伴要问了，```path```怎么修改啊，接下来我再顺便提一下mac环境变量修改的方法：
>1、打开```Terminal```
>2、输入```vim /Users/{用户名}/.bash_profile```
>3、然后vim编辑器就打开了，里面有一堆的环境变量，如图1
>4、然后输入```i```进入编辑模式
>5、然后按照文件中已有的环境变量格式输入自己之前建好的文件夹就ok啦，如图2，添加上自己的path路径
>6、最后按```esc```，输入```:```(冒号)，```wq```，然后回车保存写好的文件
>7、执行```source /Users/{用户名}/.bash_profile```，重新加载环境变量使修改生效
>8、试试你拖进去的东西是不是可以用```Terminal```打开了~~~~

![图1](http://upload-images.jianshu.io/upload_images/1648999-e79499e22360fc68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![图2](http://upload-images.jianshu.io/upload_images/1648999-8001d89d7e3c0fcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在网上也看到了用关闭```Rootless ```的方法可以实现解锁那些限制路径的修改，个人感觉非必须情况下还是不要解锁了，虽然安全问题可能涉及不到多少，但是系统级的东西我是能少修改就少修改，，胆小。。而且小白一个怕改东西改坏了就完蛋了.之前改环境变量就改坏过，还好有方法可以改回来
当然你要是想解锁权限的话也很简单：  
>重启，开机按住Command + R，以Recovery分区启动
>找到终端，打开，输入：`csrutil disable`  ，重启后再操作就可以了，操作完记得再设置为`enable`

我再啰嗦一下把环境变量问题导致```Terminal ```基本不能用的解决办法吧。

>1、在命令行中输入：```export PATH=/usr/bin:/usr/sbin:/bin:/sbin:/usr/X11R6/bin```（这样可以保证命令行命令暂时可以使用。命令执行完之后先不要关闭终端）
>2、输入：```cd ~/```（进入当前用户的home目录）
>3、输入```vim .bash_profile``` (打开后把错误的地方修改过来就OK了)，退出vim并保存文件
>4、此时在命令行中输入更新命令(命令行一直不要关):```source .bash_profile```
>5、关闭```Terminal ```再重新打开一个窗口，看一下```Terminal ```是不是又可以用啦

最后这个忘了在哪摘的了，如果发现是你写的请联系下我，我注一下出处~~~
