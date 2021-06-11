# linux下杀死进程（kill）的N种方法

http://blog.csdn.net/andy572633/article/details/7211546

首先，用ps查看进程，方法如下：

```
$ ps -ef

……
smx       1822     1  0 11:38 ?        00:00:49 gnome-terminal
smx       1823  1822  0 11:38 ?        00:00:00 gnome-pty-helper
smx       1824  1822  0 11:38 pts/0    00:00:02 bash
smx       1827     1  4 11:38 ?        00:26:28 /usr/lib/firefox-3.6.18/firefox-bin
smx       1857  1822  0 11:38 pts/1    00:00:00 bash
smx       1880  1619  0 11:38 ?        00:00:00 update-notifier
……
smx      11946  1824  0 21:41 pts/0    00:00:00 ps -ef

或者：

$ ps -aux

……

smx       1822  0.1  0.8  58484 18152 ?        Sl   11:38   0:49 gnome-terminal
smx       1823  0.0  0.0   1988   712 ?        S    11:38   0:00 gnome-pty-helper
smx       1824  0.0  0.1   6820  3776 pts/0    Ss   11:38   0:02 bash
smx       1827  4.3  5.8 398196 119568 ?       Sl   11:38  26:13 /usr/lib/firefox-3.6.18/firefox-bin
smx       1857  0.0  0.1   6688  3644 pts/1    Ss   11:38   0:00 bash
smx       1880  0.0  0.6  41536 12620 ?        S    11:38   0:00 update-notifier
……
smx      11953  0.0  0.0   2716  1064 pts/0    R+   21:42   0:00 ps -aux
```

此时如果我想杀了火狐的进程就在终端输入：

```
$ kill -s 9 1827
```

其中-s 9 制定了传递给进程的信号是９，即强制、尽快终止进程。各个终止信号及其作用见附录。

1827则是上面ps查到的火狐的PID。

简单吧，但有个问题，进程少了则无所谓，进程多了，就会觉得痛苦了，无论是ps -ef 还是ps -aux，每次都要在一大串进程信息里面查找到要杀的进程，看的眼都花了。

###### **改进１**：

把ps的查询结果通过管道给grep查找包含特定字符串的进程。管道符“|”用来隔开两个命令，管道符左边命令的输出会作为管道符右边命令的输入。

```
$ ps -ef | grep firefox
smx       1827     1  4 11:38 ?        00:27:33 /usr/lib/firefox-3.6.18/firefox-bin
smx      12029  1824  0 21:54 pts/0    00:00:00 grep --color=auto firefox
```

这次就清爽了。然后就是

```
$kill -s 9 1827
```

###### **改进２——使用pgrep**：

一看到pgrep首先会想到什么？没错，grep！pgrep的p表明了这个命令是专门用于进程查询的grep。

```
$ pgrep firefox
1827
```

看到了什么？没错火狐的PID，接下来又要打字了：

```
$kill -s 9 1827
```

###### **改进３——使用pidof：**

看到pidof想到啥？没错pid of xx，字面翻译过来就是 xx的PID。

```
$ pidof firefox-bin
1827
```


和pgrep相比稍显不足的是，pidof必须给出进程的全名。然后就是老生常谈：

```
$kill -s 9 1827
```

无论使用ps 然后慢慢查找进程PID 还是用grep查找包含相应字符串的进程，亦或者用pgrep直接查找包含相应字符串的进程ＰＩＤ，然后手动输入给ｋｉｌｌ杀掉，都稍显麻烦。有没有更方便的方法？有！

###### **改进４：**

```
$ps -ef | grep firefox | grep -v grep | cut -c 9-15 | xargs kill -s 9
```

说明：

“grep firefox”的输出结果是，所有含有关键字“firefox”的进程。

“grep -v grep”是在列出的进程中去除含有关键字“grep”的进程。

“cut -c 9-15”是截取输入行的第9个字符到第15个字符，而这正好是进程号PID。

“xargs kill -s 9”中的xargs命令是用来把前面命令的输出结果（PID）作为“kill -s 9”命令的参数，并执行该命令。“kill -s 9”会强行杀掉指定进程。

###### **改进５：**

知道pgrep和pidof两个命令，干嘛还要打那么长一串！

```
$ pgrep firefox | xargs kill -s 9
```



###### 改进６：

```
$ ps -ef | grep firefox | awk '{print $2}' | xargs kill -9
kill: No such process
```

有一个比较郁闷的地方，进程已经正确找到并且终止了，但是执行完却提示找不到进程。

其中awk '{print $2}' 的作用就是打印（print）出第二列的内容。根据常规篇，可以知道ps输出的第二列正好是PID。就把进程相应的PID通过xargs传递给kill作参数，杀掉对应的进程。

###### **改进７**：

难道每次都要调用xargs把PID传递给kill？答案是否定的：

```
$kill -s 9 `ps -aux | grep firefox | awk '{print $2}'`
```

###### **改进８：**

没错，命令依然有点长，换成pgrep。

```
$kill -s 9 `pgrep firefox`
```

###### **改进9——pkill：**

看到pkill想到了什么？没错pgrep和kill！pkill＝pgrep+kill。

```
$pkill -９ firefox
```

说明："-9" 即发送的信号是9，pkill与kill在这点的差别是：pkill无须 “ｓ”，终止信号等级直接跟在 “-“ 后面。之前我一直以为是 "-s 9"，结果每次运行都无法终止进程。

###### ***\*改进10\**——killall**：

killall和pkill是相似的,不过如果给出的进程名不完整，killall会报错。pkill或者pgrep只要给出进程名的一部分就可以终止进程。

```
$killall -9 firefox
```

