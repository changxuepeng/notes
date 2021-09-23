https://mp.weixin.qq.com/s/3W5NuLOk77DsfTKoRFixMQ

# CompletionService 

本文就来聊聊 CompletionService 这个东西。

在聊它之前，我们先回顾一下 Future 的用法。

我先问问你，当你往线程池里面提交了一组计算任务以后，你想要获得返回值。

你应该用 Executor 的什么提交方法？这个提交方法的什么重载类型？

之前的文章里面写了啊：

![图片](CompletionService.assets/640)

由于是一组计算任务，你想拿到返回值去搞事情。这个返回值就被封装在 Future 里面。

怎么获取呢？

调用 Future 的 get 方法，有不带超时时间的无限等待的舔狗类型的 get，也有带超时时间、到点就放弃的渣男类型的 get：

![图片](CompletionService.assets/640-16323601827712)

来一起看个例子吧：

```java
public class JDKThreadPoolExecutorTest {

    public static void main(String[] args) throws Exception {
        ExecutorService executorService = Executors.newCachedThreadPool();
        ArrayList<Future<Integer>> list = new ArrayList<>();
        Future<Integer> future_15 = executorService.submit(() -> {
            TimeUnit.SECONDS.sleep(15);
            System.out.println("执行时长为15s的执行完成。");
            return 15;
        });
        list.add(future_15);
        
        Future<Integer> future_5 = executorService.submit(() -> {
            TimeUnit.SECONDS.sleep(5);
            System.out.println("执行时长为5s的执行完成。");
            return 5;
        });
        list.add(future_5);
        
        Future<Integer> future_10 = executorService.submit(() -> {
            TimeUnit.SECONDS.sleep(10);
            System.out.println("执行时长为10s的执行完成。");
            return 10;
        });
        list.add(future_10);
        
        System.out.println("开始准备获取结果");
        for (Future<Integer> future : list) {
            System.out.println("future.get() = " + future.get());
        }
        Thread.currentThread().join();
    }
}
```

现在有三个任务，执行时间分别是 15s/10s/5s 。通过 JDK 线程池的 submit 方法提交了这三个 Callable 类型的任务。

你先眼神编译一下，心里输出一下，你想这个代码的输出结果是什么。

首先主线程把三个任务提交到线程池里面去，把对应返回的 Future 放到 List 里面存起来，然后执行“开始准备获取结果”的输出语句。

接着进入 for 循环，在循环里面执行 future.get() 操作，阻塞等待。

看看你心里想的输出结果是不是这样的：

![图片](CompletionService.assets/640-16323602454114)

从这个输出结果里面，我们可以看出问题了。很明显的木桶效应。

三个异步任务，耗时最长的最先执行，所以最先进入 list，因此当在循环中获取这个任务结果的时候 get 操作会一直阻塞，即使执行时间为 5s/10s 的任务已经执行完成。

好的，举个例子。想象一个场景：

假设你是一个海王，你拥有众多普通女性朋友。你同时邀约了三位女性朋友一起吃饭。分别给她们说：你先化妆吧，好了给我说一声，我开车来接你。

小红化妆要 2 小时。小花化妆要 1小时。小媛化妆要 30 分钟。

由于你最先给小红说的，你就一直在小红家门口等小红化妆完成。当小红化妆完成后，你接到车上，其他两位朋友早就准备好了，在家干巴巴的等着你来接她。

这不是一个合格的海王应该有的样子。

这就是 future 在这种场景下的局限性。

根据上面的场景编码可得（代码都是直接复制粘贴就可以用的，建议你拿出来跑一下）：

```java
public class JDKThreadPoolExecutorTest {

    public static void main(String[] args) throws Exception {
        ExecutorService executorService = Executors.newCachedThreadPool();
        ArrayList<Future<String>> list = new ArrayList<>();
        System.out.println("约几个妹子一起吃个饭吧。");
        Future<String> future_15 = executorService.submit(() -> {
            System.out.println("小红：好的，哥哥。我化妆要2个小时。等一下哦。");
            TimeUnit.SECONDS.sleep(15);
            System.out.println("小红：我2个小时准时化好了，哥哥来接我吧。");
            return "小红化完了。";
        });
        list.add(future_15);
        Future<String> future_5 = executorService.submit(() -> {
            System.out.println("小媛：好的，哥哥。我化妆要30分钟。等一下哦。");
            TimeUnit.SECONDS.sleep(5);
            System.out.println("小媛：我30分钟准时化好了，哥哥来接我吧。");
            return "小媛化完了。";
        });
        list.add(future_5);

        Future<String> future_10 = executorService.submit(() -> {
            System.out.println("小花：好的，哥哥。我化妆要1个小时。等一下哦。");
            TimeUnit.SECONDS.sleep(10);
            System.out.println("小花：我1个小时准时化好了，哥哥来接我吧。");
            return "小花化完了。";
        });
        list.add(future_10);
        TimeUnit.SECONDS.sleep(1);
        System.out.println("都通知完,等着吧。");
          for (Future<String> future : list) {
            System.out.println(future.get()+"我去接她。");
        }
        Thread.currentThread().join();
    }
}
```

输出结果如下：

![图片](CompletionService.assets/640-16323603069136)

说好都是一样的普通朋友的，为什么你偏偏要一直等化妆时间最长的小红？为什么不谁动作快，就先接谁？

你看你这样操作，让小媛、小花怎么想？只能说：你是一个好人了。

## CompletionService拯救海王

还是上面的场景，当我们引入了 CompletionService 后就显得不一样了。

先直接看用法：

```java
ExecutorService executorService = Executors.newCachedThreadPool();
ExecutorCompletionService<String> completionService = new ExecutorCompletionService<>(executorService);
```

用起来非常的方便，只需要用 ExecutorCompletionService 把线程池包起来。

然后提交任务的时候用 competitionService 的 submit 方法。代码如下：

```java
public class ExecutorCompletionServiceTest {

    public static void main(String[] args) throws Exception {
        ExecutorService executorService = Executors.newCachedThreadPool();
        ExecutorCompletionService<String> completionService =
                new ExecutorCompletionService<>(executorService);
        System.out.println("约几个妹子一起吃个饭吧。");
        completionService.submit(() -> {
            System.out.println("小红：好的，哥哥。我化妆要2个小时。等一下哦。");
            TimeUnit.SECONDS.sleep(15);
            System.out.println("小红：我2个小时准时化好了，哥哥来接我吧。");
            return "小红化完了。";
        });
        completionService.submit(() -> {
            System.out.println("小媛：好的，哥哥。我化妆要30分钟。等一下哦。");
            TimeUnit.SECONDS.sleep(5);
            System.out.println("小媛：我30分钟准时化好了，哥哥来接我吧。");
            return "小媛化完了。";
        });
        completionService.submit(() -> {
            System.out.println("小花：好的，哥哥。我化妆要1个小时。等一下哦。");
            TimeUnit.SECONDS.sleep(10);
            System.out.println("小花：我1个小时准时化好了，哥哥来接我吧。");
            return "小花化完了。";
        });
        TimeUnit.SECONDS.sleep(1);
        System.out.println("都通知完,等着吧。");
        //循环3次是因为上面提交了3个异步任务
        for (int i = 0; i < 3; i++) {
            String returnStr = completionService.take().get();
            System.out.println(returnStr + "我去接她");
        }
        Thread.currentThread().join();
    }
}
```

你先眼神编译一下，心里输出一下...

算了，别编译了，直接带大家看结果吧，我已经迫不及待了：

![图片](CompletionService.assets/640-16323603611188)

谁先化完妆，就先去接谁。

写到这里，看到这个输出结果的时候我不禁鼓起掌来。

真正的海王应该是一个时间管理大师。

先对比一下输出结果，全体起立，一起鼓掌：

![图片](CompletionService.assets/640-163236037197910)

然后对比一下两个版本代码的差异：

![图片](CompletionService.assets/640-163236037367912)

变化不大，甚至说微乎其微。

执行 submit 方法的对象变成了 ExecutorCompletionService 。

获取任务结果的方法变成了：

> String returnStr = completionService.take().get();

先不看原理。你就细细的品这个获取结果的方法。

completionService.take() 了个什么玩意出来，然后调用了 get 方法。

根据这个 get ，直觉就告诉我 take 出来的肯定是一个 future 对象。而这个 future 对象肯定是放在一个队列里面的。

下一小节，带大家去证实一下。

首先 CompletionService 是一个接口：

![图片](CompletionService.assets/640-163236039144414)

ExecutorCompletionService 是这个接口的实现类：

![图片](CompletionService.assets/640-163236040170616)

看一下 ExecutorCompletionService 的构造方法：

![图片](CompletionService.assets/640-163236040342818)

可以看到是需要传入一个线程池对象的。队列默认使用的是 LinkedBlockingQueue 。

当然，我们也可以指定使用什么队列：

![图片](CompletionService.assets/640-163236045390320)

然后再看一下它的任务提交方式：

![图片](CompletionService.assets/640-163236046161922)

由于用 ExecutorCompletionService 主要是为了优雅的处理返回值。所以它支持两种 submit 类型的提交，都是有返回值的。

上面时间管理大师版本海王使用的就是 Callable 类型的方法。

我们先对比一下 Executor 直接提交和 ExecutorCompletionService 提交的差异：

![图片](CompletionService.assets/640-163236048640624)

差异就在 execute 方法里面。

ExecutorCompletionService 提交任务的时候是这样的：

> executor.execute(new QueueingFuture(f));

差异就在 execute 方法里面的 Runable：

![图片](CompletionService.assets/640-163236053321826)

看一下这个 QueueingFuture 是个什么东西：

![图片](CompletionService.assets/640-163236053549028)

秘密基本上就在这个里面了。

QueueingFuture 继承自 FutureTask。重写了 done 方法，然后把 task 放到 queue 里面。

这个方法的含义就是当任务执行完成后，就会被放到队列里面去了。也就是说队列里面的 task 都是已经 done 了的 task，而这个 task 就是一个个 future。

如果调用 queue 的 take 方法，就是阻塞等待。等到的一定是就绪了的 future，调用 get 就能立马获得结果。

你说这一套操作是在干啥？

这不就是在做解耦吗？

之前你提交任务后还需要直接关心每个任务返回的 future。现在 CompletionService 帮你对这些 future 进行了跟踪。

**完成了调用者和 future 之间的解耦。**

原理分析完了，说一个需要注意的地方。

**当你的使用场景是不关心返回值的时候千万不要闲的蛋疼的用 CompletionService 去提交任务。**

为什么？

因为前面说了，里面有个队列。而当你不关心返回值的时候也就是不会去处理这个队列，导致这个队列里面的对象堆积的越来越多。

最后，炸了，OOM了。

## 在开源框架中的应用

前面说了 CompletionService 是一个接口。除了 JDK 的 ExecutorCompletionService 实现了这个接口。

在开源框架里面也有相应的实现。比如 Redisson：

![图片](CompletionService.assets/640-163236075549230)

你去看这个实现，和 ExecutorCompletionService 思想是一模一样的，但是实现是有些许的不一样。

它把 future 放到队列面的时候，没有重写 done 方法，而是使用了响应式编程的 onComplete：

![图片](CompletionService.assets/640-163236075939432)

而 CompletionService 的思想核心是：Executor 加 Queue 。

这个思想，让我想起了在 Dubbo 中看到过的一个类：

![图片](CompletionService.assets/640-163236080706938)

我曾经在[《Dubbo Cluster集群那点你不知道的事》](https://mp.weixin.qq.com/s?__biz=Mzg3NjU3NTkwMQ==&mid=2247505092&idx=1&sn=8896647a6b7959a2f619626dfcdbb2e9&source=41&scene=21#wechat_redirect)这篇文章中提到过这个类。

![图片](CompletionService.assets/640-163236080358536)

这个类的 doInvoker 方法中的核心逻辑如下：

![图片](CompletionService.assets/640-163236080063434)

首先标号为 ① 的地方定义了一个队列。

标号为 ② 的地方在循环体中提交异步任务。需要几个服务提供者就有几次循环。

子线程在标号为 ③ 的地方把返回结果放到队列里面。

只要一放进去，就能被标号为 ④ 的地方获取到（指定时间内），然后程序立即返回。

这样就能实现并行调用多个服务提供者，只要有一个服务提供者返回就立即返回的功能。

我觉得这个思想和 CompletionService 的思想有一点点的相通之处的。

我们要学 CompletionService ，也要学它的思想。