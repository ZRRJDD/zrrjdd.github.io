
# 1.关于Mac OS修改 .bash_profile后无法生效问题。

当我们在修改.bash_profile文件后，发现每次都需要 `source .bash_profile`才可以生效，但是退出终端后就失效了。这时候 有可能是mac 默认shell的问题。

可以先修改用户的 shell，使用命令,来更换shell：
```shell
chsh -s /bin/bash
```

- [Mac官网 Mac上将zsh用作默认Shell](https://support.apple.com/zh-cn/HT208050)
- [参考文档](https://blog.csdn.net/weixin_44781205/article/details/90369940)
- [mac 装了 oh my zsh 后比用 bash 具体好在哪儿？](https://www.zhihu.com/question/29977255)
- [Mac下切换zsh和bash](https://www.jianshu.com/p/8d822ce0d425)