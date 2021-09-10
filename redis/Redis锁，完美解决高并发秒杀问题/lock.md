# 借助Redis锁，完美解决高并发秒杀问题

https://mp.weixin.qq.com/s/bBPUFBY24JAJVQj06csJzw

场景：一家网上商城做商品限量秒杀。

## **1 单机环境下的锁**

将商品的数量存到Redis中。每个用户抢购前都需要到Redis中查询商品数量（代替mysql数据库。不考虑事务），如果商品数量大于0，则证明商品有库存。然后我们在进行库存扣减和接下来的操作。

因为多线程并发问题，我们不得不在get()方法内部使用同步代码块。这样可以保证查询库存和减库存操作的原子性。

```java
package springbootdemo.demo.controller;
/*
 * @auther 顶风少年
 * @mail dfsn19970313@foxmail.com
 * @date 2020-01-13 11:19
 * @notify
 * @version 1.0
 */

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RedisLock  {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @GetMapping(value = "buy")
    public String get() {
        synchronized (this) {
            String phone = redisTemplate.opsForValue().get("phone");
            Integer count = Integer.valueOf(phone);
            if (count > 0) {
                redisTemplate.opsForValue().set("phone", String.valueOf(count - 1));
                System.out.println("抢到了" + count + "号商品");
            }return "";
        }
    }
}
```

## **2 分布式情况下使用Redis锁。**

但是由于业务上升，并发数量变大。公司不得不将原有系统复制一份，放到新的服务器。然后使用nginx做负载均衡。为了模拟高并发环境这里使用了 Apache JMeter工具。

很明显，现在的线程锁不管用了。于是我们需要换一把锁，这把锁必须和两套系统没有任何的耦合度。

使用Redies的API如果key不存在，则设置一个key。这个key就是我们现在使用的一把锁。每个线程到此处，先设置锁，如果设置锁失败，则表明当前有线程获取到了锁，就返回。

最后我们为了减库存和其他业务抛出异常，而没有释放锁。把释放锁的操作放到了finally代码块中。看起来是比较完美了。

```java
package springbootdemo.demo.controller;
/*
 * @auther 顶风少年
 * @mail dfsn19970313@foxmail.com
 * @date 2020-01-13 11:19
 * @notify
 * @version 1.0
 */

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RedisLock {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @GetMapping(value = "buy")
    public String get() {
        Boolean phoneLock = redisTemplate.opsForValue().setIfAbsent("phoneLock", "");
        if (!phoneLock) {
            return "";
        }
        try{
            String phone = redisTemplate.opsForValue().get("phone");
            Integer count = Integer.valueOf(phone);
            if (count > 0) {
                redisTemplate.opsForValue().set("phone", String.valueOf(count - 1));
                System.out.println("抢到了" + count + "号商品");
            }
        }finally {
            redisTemplate.delete("phoneLock");
        }
        return "";
    }
}
```

## **3 一台服务宕机，导致无法释放锁**

如果try中抛出了异常，进入finally,这把锁依然会释放，不会影响其他线程获取锁，那么如果在finally也抛出了异常，或者在finally中服务直接关闭了，那其他的服务再也获取不到锁。最终导致商品卖不出去。

```java
package springbootdemo.demo.controller;
/*
 * @auther 顶风少年
 * @mail dfsn19970313@foxmail.com
 * @date 2020-01-13 11:19
 * @notify
 * @version 1.0
 */

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RedisLock {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @GetMapping(value = "buy")
    public String get() {
        int i = 0;
        Boolean phoneLock = redisTemplate.opsForValue().setIfAbsent("phoneLock", "");
        if (!phoneLock) {
            return "";
        }
        try {
            String phone = redisTemplate.opsForValue().get("phone");
            Integer count = Integer.valueOf(phone);
            if (count > 0) {
                i = count;
                redisTemplate.opsForValue().set("phone", String.valueOf(count - 1));
                System.out.println("抢到了" + count + "号商品");
            }
        } finally {
            if (i == 20) {
                System.exit(0);
            }
            redisTemplate.delete("phoneLock");
        }
        return "";
    }
}
```

## **4 给每一把锁加上过期时间**

问题就出现在如果出现意外，这把锁无法释放。这里我们在引入Redis的API，对key进行过期时间的设置。这样如果拿到锁的线程，在任何情况下没有来得及释放锁，当Redis的key时间到，也会自动释放锁。但是这样还是存在问题

如果在key过期后，锁释放了，但是当前线程没有执行完毕。那么其他线程就会拿到锁，继续抢购商品，而这个较慢的线程则会在执行完毕后，释放别人的锁。导致锁失效!

```java
package springbootdemo.demo.controller;
/*
 * @auther 顶风少年
 * @mail dfsn19970313@foxmail.com
 * @date 2020-01-13 11:19
 * @notify
 * @version 1.0
 */

import javafx.concurrent.Task;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Timer;
import java.util.TimerTask;
import java.util.concurrent.TimeUnit;

@RestController
public class RedisLock {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @GetMapping(value = "buy")
    public String get() {
        Boolean phoneLock = redisTemplate.opsForValue().setIfAbsent("phoneLock", "", 3, TimeUnit.SECONDS);
        if (!phoneLock) {
            return "";
        }
        try {
            String phone = redisTemplate.opsForValue().get("phone");
            Integer count = Integer.valueOf(phone);
            if (count > 0) {
                try {
                    Thread.sleep(99999999999L);
                } catch (Exception e) {

                }
                redisTemplate.opsForValue().set("phone", String.valueOf(count - 1));
                System.out.println("抢到了" + count + "号商品");
            }
        } finally {
          
            redisTemplate.delete("phoneLock");
        }
        return "";
    }
}
```

## **5 延长锁的过期时间，解决锁失效**

问题的出现就是，当一条线程的key已经过期，但是这个线程的任务确确实实没有执行完毕，这个交易没有结束。但是锁没了。现在我们必须对锁的时间进行延长。在判断商品有库存时，第一时间创建一个线程不停的给key续命

防止key过期。然后在交易结束后，停止定时器，释放锁。

```java
package springbootdemo.demo.controller;
/*
 * @auther 顶风少年
 * @mail dfsn19970313@foxmail.com
 * @date 2020-01-13 11:19
 * @notify
 * @version 1.0
 */

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Timer;
import java.util.TimerTask;
import java.util.concurrent.TimeUnit;

@RestController
public class RedisLock {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @GetMapping(value = "buy")
    public String get() {
        Boolean phoneLock = redisTemplate.opsForValue().setIfAbsent("phoneLock", "", 3, TimeUnit.SECONDS);
        if (!phoneLock) {
            return "";
        }
        Timer timer = null;
        try {
            String phone = redisTemplate.opsForValue().get("phone");
            Integer count = Integer.valueOf(phone);
            if (count > 0) {
                timer = new Timer();
                timer.schedule(new TimerTask() {
                    @Override
                    public void run() {
                        redisTemplate.opsForValue().set("phoneLock", "", 3, TimeUnit.SECONDS);
                    }
                }, 0, 1);

                redisTemplate.opsForValue().set("phone", String.valueOf(count - 1));
                System.out.println("抢到了" + count + "号商品");
            }
        } finally {
            if (timer != null) {
                timer.cancel();
            }
            redisTemplate.delete("phoneLock");
        }
        return "";
    }
}
```

## **6 使用Redisson简化代码**

在步骤5我们的代码已经很完善了，不会出现高并发问题。但是代码确过于冗余，我们为了使用Redis锁，我们需要设置一个定长的key，然后当购买完成后，将key删除。但为了防止key提前过期，我们不得不新增一个线程执行定时任务。

下面我们可以使用Redissson框架简化代码。`getLock()`方法代替了Redis的`setIfAbsent()`，`lock()`设置过期时间。最终我们在交易结束后释放锁。延长锁的操作则有Redisson框架替我们完成，它会使用轮询去查看key是否过期，

在交易没有完成时，自动重设Redis的key过期时间

```java
package springbootdemo.demo.controller;
/*
 * @auther 顶风少年
 * @mail dfsn19970313@foxmail.com
 * @date 2020-01-13 11:19
 * @notify
 * @version 1.0
 */

import org.redisson.Redisson;
import org.redisson.api.RLock;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Timer;
import java.util.TimerTask;
import java.util.concurrent.TimeUnit;

@RestController
public class RedissonLock {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Autowired
    private Redisson redisson;

    @GetMapping(value = "buy2")
    public String get() {
        RLock phoneLock = redisson.getLock("phoneLock");
        phoneLock.lock(3, TimeUnit.SECONDS);
        try {
            String phone = redisTemplate.opsForValue().get("phone");
            Integer count = Integer.valueOf(phone);
            if (count > 0) {
                redisTemplate.opsForValue().set("phone", String.valueOf(count - 1));
                System.out.println("抢到了" + count + "号商品");
            }
        } finally {
            phoneLock.unlock();
        }
        return "";
    }
}
```

