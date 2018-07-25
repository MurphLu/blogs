为了改变一下终端默认比较丑的界面，最近装了下oh My zsh这个神器，装完使用一段时间后发现确实很强大，但最近发现有些命令不能用，遂google一通，最后找到解决办法：
修改 .bash_profile
进入终端，输入`vim ~/.bash_profile`
添加下面一行` export PATH=/bin:/usr/bin:/usr/local/bin:"${PATH}"`
保存退出vim，
输入 `source ~/.bash_profile`
然后再试一下不能正常使用的命令，是不是已经好了呢~~~

参考：http://stackoverflow.com/questions/18428374/commands-not-found-on-zsh

2017-3-27————更新
最近更新完之后又一次抽风，查了查最后在~/.zshrc中加入了`source ~/.bash_profile`这一行，到现在还依然正常，不知道会不会哪天又一次挂掉
