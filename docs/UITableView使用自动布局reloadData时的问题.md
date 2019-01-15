今天撸代码的时候遇到这样一个问题：
问题出现条件：UITableViewCell使用自动布局
当我加载tableView中第二页及后面页时，获取到数据后调用reloadData方法时会出现tableView数据往上移一段位置，大概情况如下图：

![estimatedHeight设置为100时.gif](img/0008-01.gif)

查了好半天最后发现是应为`estimatedHeight`设置的有问题，如果已经加载的Cell有比`estimatedHeight`的高度高的情况下就会有这个问题出现，所以`estimatedHeight`最好是设置一个比Cell可能出现的最大高度大的一个值，这样的话就不会有这样的问题出现。


![estimatedHeight设置为400时.gif](img/0008-02.gif)

具体加载过程想了一下想了个七七八八，但是感觉还不能太准确的表达，等我整理整理再加，这里就先记下这个问题，也希望亲们能够给些指导~~
