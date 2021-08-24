## 死锁的 4 种排查工具 ！

https://mp.weixin.qq.com/s/NJOBBdy8C26pNLmgnUODbA

## 定义

死锁（Dead Lock）指的是两个或两个以上的运算单元（进程、线程或协程），都在等待对方停止执行，以取得系统资源，但是没有一方提前退出，就称为死锁。

![图片](lock.assets/1)

## 死锁示例

接下来，我们先来演示一下 Java 中最简单的死锁，我们创建两个锁和两个线程，让线程 1 先拥有锁 A，然后在 1s 后尝试获取锁  B，同时我们启动线程 2，让它先拥有锁 B，然后在 1s 之后尝试获取锁  A，这时就会出现相互等待对方释放锁的情况，从而造成死锁的问题，具体代码如下：

```java
public class DeadLockExample {
    public static void main(String[] args) {
        Object lockA = new Object(); // 创建锁 A
        Object lockB = new Object(); // 创建锁 B

        // 创建线程 1
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                // 先获取锁 A
                synchronized (lockA) {
                    System.out.println("线程 1:获取到锁 A!");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 尝试获取锁 B
                    System.out.println("线程 1:等待获取 B...");
                    synchronized (lockB) {
                        System.out.println("线程 1:获取到锁 B!");
                    }
                }
            }
        });
        t1.start(); // 运行线程

        // 创建线程 2
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                // 先获取锁 B
                synchronized (lockB) {
                    System.out.println("线程 2:获取到锁 B!");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 尝试获取锁 A
                    System.out.println("线程 2:等待获取 A...");
                    synchronized (lockA) {
                        System.out.println("线程 2:获取到锁 A!");
                    }
                }
            }
        });
        t2.start(); // 运行线程
    }
}
```

以上程序的执行结果如下：

<img src="lock.assets/2" alt="图片" style="zoom:67%;" />

从上述结果可以看出，线程 1 和线程 2 都在等待对方释放锁，这样就造成了死锁问题。

## 死锁产生原因

通过以上示例，我们可以得出结论，要产生**死锁需要满足以下 4 个条件**：

**互斥条件**：指运算单元（进程、线程或协程）对所分配到的资源具有排它性，也就是说在一段时间内某个锁资源只能被一个运算单元所占用。

**请求和保持条件**：指运算单元已经保持至少一个资源，但又提出了新的资源请求，而该资源已被其它运算单元占有，此时请求运算单元阻塞，但又对自己已获得的其它资源保持不放。

**不可剥夺条件**：指运算单元已获得的资源，在未使用完之前，不能被剥夺。

**环路等待条件**：指在发生死锁时，必然存在运算单元和资源的环形链，即运算单元正在等待另一个运算单元占用的资源，而对方又在等待自己占用的资源，从而造成环路等待的情况。

只有以上 4 个条件同时满足，才会造成死锁问题。

## 死锁排查

如果程序出现死锁问题，可通过以下 4 种方案中的任意一种进行分析和排查。

### **方案 1：jstack**

我们在使用 jstack 之前，先要通过 jps 得到运行程序的进程 ID，使用方法如下：

![图片](lock.assets/3)

“jps -l”可以查询本机所有的 Java 程序，jps（Java Virtual Machine Process Status Tool）是  Java 提供的一个显示当前所有 Java 进程 pid 的命令，适合在 linux/unix/windows 平台上简单查看当前 Java  进程的一些简单情况，“-l”用于输出进程 pid 和运行程序完整路径名（包名和类名）。

有了进程 ID（PID）之后，我们就可以使用“jstack -l PID”来发现死锁问题了，如下图所示：

![图片](lock.assets/4)

jstack 用于生成 Java 虚拟机当前时刻的线程快照，“-l”表示长列表（long），打印关于锁的附加信息。

> PS：可以使用 jstack -help 查看更多命令使用说明。

### **方案 2：jconsole**

![图片](lock.assets/5)

然后选择要调试的程序，如下图所示：

![图片](lock.assets/6)

之后点击连接进入，选择“不安全的连接”进入监控主页，如下图所示：

<img src="lock.assets/7" alt="图片" style="zoom:67%;" />

<img src="lock.assets/8" alt="图片" style="zoom:67%;" />

之后切换到“线程”模块，点击“检测死锁”按钮，如下图所示：

<img src="lock.assets/9" alt="图片" style="zoom:67%;" />

之后稍等片刻就会检测出死锁的相关信息，如下图所示：

<img src="lock.assets/10" alt="图片" style="zoom:67%;" />

### **方案 3：jvisualvm**

<img src="lock.assets/11" alt="图片" style="zoom:67%;" />

稍等几秒之后，jvisualvm 中就会出现本地的所有 Java 程序，如下图所示：

![图片](lock.assets/12)

双击选择要调试的程序：

![图片](lock.assets/13)

单击鼠标进入“线程”模块，如下图所示：

![图片](lock.assets/14)

从上图可以看出，当我们切换到线程一栏之后就会直接显示出死锁信息，之后点击“线程 Dump”生成死锁的详情信息，如下图所示：

![图片](lock.assets/15)

### **方案 4：jmc**

jmc 是 Oracle Java Mission Control 的缩写，是一个对 Java 程序进行管理、监控、概要分析和故障排查的工具套件。它也是在 JDK 的 bin 目录中，同样是双击启动，如下图所示

![图片](lock.assets/16)

jmc 主页信息如下：

![图片](lock.assets/17)

之后选中要排查的程序，右键“启动 JMX 控制台”查看此程序的详细内容，如下图所示：

<img src="lock.assets/18" alt="图片" style="zoom: 50%;" />

![图片](lock.assets/19)

然后点击“线程”，勾中“死锁检测”就可以发现死锁和死锁的详情信息，如下图所示：

![图片](lock.assets/20)

## 总结

死锁是因为两个或两个以上的运算单元，都在等待对方停止执行，以取得系统资源，但没有一方提前退出，于是就出现了死锁。死锁的排查工具总共有 4 种：

- jstack
- jconsole
- jvisualvm
- jmc

从易用性和性能方面来考虑，推荐使用 jconsole 或 jvisualvm 来排查死锁。