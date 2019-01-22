Max OS X11之后在内核下引入了Rootless机制，以下路径

```
/System
/bin
/sbin
/usr (except /usr/local)
```

即使是root用户也不能随意修改，然后在执行`gem install` 的时候即使使用 `sudo` 也是没有权限的，当然可以解锁Rootless但是为了安全可以将 `gem install` 改为其他 `sudo` 可以访问的路径 例如 `/usr/local/bin/`
然后在执行 `gem install` 时可以使用 `-n` 来执行安装路径，拿`fastlane`来说，可以使用
`sudo gem install fastlane -n /usr/local/bin` 来执行安装