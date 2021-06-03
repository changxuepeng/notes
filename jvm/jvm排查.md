https://mp.weixin.qq.com/s/UkATFZHQzrIERZK5OaT74g

内存分析工具mat 下载地址

 https://www.eclipse.org/mat/

#### 内存泄漏

于是赶快登陆探测服务器，首先是 `top free df` 三连，结果还真发现了些异常。

1.使用 `jstat -gc pid [interval]` 命令查看了 java 进程的 GC 状态

2.使用 `jstack pid > jstack.log` 保存了线程栈的现场，

3.使用 `jmap -dump:format=b,file=heap.log pid` 保存了堆现场



#### jstat

jstat 是一个非常强大的 JVM 监控工具，一般用法是： `jstat [-options] pid interval`

它支持的查看项有：

- -class 查看类加载信息

- -compile 编译统计信息

- -gc 垃圾回收信息

- -gcXXX 各区域 GC 的详细信息 如 -gcold

  

#### 分析栈

```
grep 'java.lang.Thread.State' jstack.log  | wc -l

grep -A 1 'java.lang.Thread.State' jstack.log  | grep -v 'java.lang.Thread.State' | sort | uniq -c |sort -n

```





#### 下载堆 dump 文件

`gzip` 是个功能很强大的压缩命令，特别是我们可以设置 `-1 ~ -9` 来指定它的压缩级别，数据越大压缩比率越大，耗时也就越长，推荐使用 -6~7

#### 使用 MAT 分析 jvm heap



MAT 是分析 Java 堆内存的利器，使用它打开我们的堆文件（将文件后缀改为 `.hprof`）, 它会提示我们要分析的种类，对于这次分析，果断选择 `memory leak suspect`