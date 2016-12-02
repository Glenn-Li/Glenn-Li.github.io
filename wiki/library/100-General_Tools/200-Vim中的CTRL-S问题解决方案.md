
# VIM 中Ctrl-S把屏幕挂死的解决方法

**来源 http://www.thinkinsth.net/blog/354**

Ctrl-S 可能算是 Windows 下最常用的保存快捷键了，但是在类 Unix 系统下面却不是起保存的功能。

最近都是在用 Cygwin + VIM 阅读和编写代码，时不时会不注意按到 Ctrl-S，导致 VIM 跟挂住了一样，啥都无法干，最后只能重启 shell，一开始我以为是 VIM 在Cygwin 下面有啥不适应的，所以就在 ~/.vimrc 里面把 Ctrl-S 重定义为保存文件的快捷键，不料还是不行。

今天终于受不了就 Google 了一把，才知道 Ctrl-S 原来在 Linux 下是用于停止显示终端的输出用的，这个时候屏幕就像定住了一样，终端不会继续输出，也不会响应你的输入，所以不知道的时候会以为终端挂起了。其实只需要按一下 **Ctrl-Q** 即可恢复输入。

这个功能可以用于长时间编译的时候随时把屏幕输出停住，但是编译仍然会继续，这样就不会影响到编译时间了。VIM doc 里面关于这个问题的描述如下：

```txt
32.1. I am running Vim in a xterm. When I press the CTRL-S key, Vim
freezes. What should I do now?
Many terminal emulators and real terminal drivers use the CTRL-S key to
stop the data from arriving so that you can stop a fast scrolling display
to look at it (also allowed older terminals to slow down the computer so
that it did not get buffer overflows). You can start the output again by
pressing the CTRL-Q key.
When you press the CTRL-S key, the terminal driver will stop sending the
output data. As a result of this, it will look like Vim is hung. If you
press the CTRL-Q key, then everything will be back to normal.
You can turn of the terminal driver flow control using the ’stty’ command:
$ stty -ixon -xoff
or, you can change the keys used for the terminal flow control, using the
following commands:
$ stty stop
$ stty start
```
