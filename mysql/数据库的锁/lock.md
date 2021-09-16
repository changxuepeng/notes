# 数据库的锁

https://mp.weixin.qq.com/s/fmSHG0SejfD0IdnpIYHT9w

MySQL Innodb的锁的相关介绍，在MySQL的官方文档(https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-insert-intention-locks )中有一定的介绍，本文的介绍也是基于这篇官方文档的。



### Record Lock

**Record Lock，翻译成记录锁，是加在索引记录上的锁。**例如，`SELECT c1 FROM t WHERE c1 = 10 For UPDATE;`会对c1=10这条记录加锁，为了防止任何其他事务插入、更新或删除c1值为10的行。

![图片](lock.assets/640)

需要特别注意的是，**记录锁锁定的是索引记录**。即使表没有定义索引，InnoDB也会创建一个隐藏的聚集索引，并使用这个索引来锁定记录。

### **Gap Lock**

**Gap Lock，翻译成间隙锁，他指的是在索引记录之间的间隙上的锁**，或者在第一个索引记录之前或最后一个索引记录之后的间隙上的锁。

那么，这里所谓的Gap（间隙）又怎么理解呢？

**Gap指的是InnoDB的索引数据结构中可以插入新值的位置。**

当你用语句SELECT…FOR UPDATE锁定一组行时。InnoDB可以创建锁，应用于索引中的实际值以及他们之间的间隙。例如，如果选择所有大于10的值进行更新，间隙锁将阻止另一个事务插入大于10的新值。

![图片](lock.assets/640-16317535983412)

**既然是锁，那么就可能会影响到数据库的并发性**，所以，间隙锁只有在Repeatable Reads这种隔离级别中才会起作用。

在Repeatable Reads这种隔离下，对于锁定的读操作（select … for update 、 lock in share mode)、update操作、delete操作时，会进行如下的加锁：

- 对于具有唯一搜索条件的唯一索引，InnoDB只锁定找到的索引记录，而不会锁定间隙。
- 对于其他搜索条件，InnoDB锁定扫描的索引范围，使用gap lock或next-key lock来阻塞其他事务插入范围覆盖的间隙。

**也就是说，对于SELECT FOR UPDATE、LOCK IN SHARE MODE、UPDATE和DELETE等语句处理时，除了对唯一索引的唯一搜索外都会获取gap锁或next-key锁，即锁住其扫描的范围。**

### **Next-Key Lock**

Next-Key锁是索引记录上的记录锁和索引记录之前间隙上的间隙锁的组合。

![图片](lock.assets/640-16317537327954)

假设一个索引包含值10、11、13和20。此索引可能的next-key锁包括以下区间:

```
(-∞, 10]

(10, 11]

(11, 13]

(13, 20]

(20, ∞ ]
```

对于最后一个间隙，∞不是一个真正的索引记录，因此，实际上，这个next-key锁只锁定最大索引值之后的间隙。

所以，Next-Key 的锁的范围都是左开右闭的。

Next-Key Lock和Gap Lock一样，只有在InnoDB的RR隔离级别中才会生效。

### **Repeatable Reads能解决幻读**

很多人看过网上的关于数据库事务级别的介绍，会认为MySQL中Repeatable Reads能解决不可重复读的问题，但是不能解决幻读，只有Serializable才能解决。但其实，这种想法是不对的。

因为MySQL跟标准RR不一样，标准的Repeatable Reads确实存在幻读问题，**但InnoDB中的Repeatable Reads是通过next-key lock解决了RR的幻读问题的**。

因为我们知道，因为有了next-key lock，所以在需要加行锁的时候，会同时在索引的间隙中加锁，这就使得其他事务无法在这些间隙中插入记录，这就解决了幻读的问题。

关于这个问题，引起过广泛的讨论，可以参考：https://github.com/Yhzhtk/note/issues/42 ，这里有很多大神发表过自己的看法。

### **MySQL的加锁原则**

前面介绍过了Record Lock、Gap Lock和Next-Key Lock，但是并没有说明加锁规则。关于加锁规则，我是看了丁奇大佬的《MySQL实战45讲》中的文章之后理解的，他总结的加锁规则里面，包含了两个“原则”、两个“优化”和一个“bug”：

原则 1：加锁的基本单位是 next-key lock。是一个前开后闭区间。原则 2：查找过程中访问到的对象才会加锁。优化 1：索引上的等值查询，给唯一索引加锁的时候，next-key lock 退化为行锁。优化 2：索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，next-key lock 退化为间隙锁。一个 bug：唯一索引上的范围查询会访问到不满足条件的第一个值为止。

假如，数据库表中当前有以下记录：

![图片](lock.assets/640-16317539677546)

当我们执行`update t set d=d+1 where id = 7`的时候，由于表 t 中没有 id=7 的记录，所以：

- 根据原则 1，加锁单位是 next-key lock，session A 加锁范围就是 (5,10]；
- 根据优化 2，这是一个等值查询 (id=7)，而 id=10 不满足查询条件，next-key lock 退化成间隙锁，因此最终加锁的范围是 (5,10)。

当我们执行`select * from t where id>=10 and id<11 for update`的时候：

- 根据原则 1，加锁单位是 next-key lock，会给 (5,10]加上 next-key lock，范围查找就往后继续找，找到 id=15 这一行停下来
- 根据优化 1，主键 id 上的等值条件，退化成行锁，只加了 id=10 这一行的行锁。
- 根据原则 2，访问到的都要加锁，因此需要加 next-key lock(10,15]。因此最终加的是行锁 id=10 和 next-key lock(10,15]。

当我们执行`select * from t where id>10 and id<=15 for update`的时候：* 根据原则 1，加锁单位是 next-key lock，会给 (10,15]加上 next-key lock，并且因为 id 是唯一键，所以循环判断到 id=15 这一行就应该停止了。* 但是，InnoDB 会往前扫描到第一个不满足条件的行为止，也就是 id=20。而且由于这是个范围扫描，因此索引 id 上的 (15,20]这个 next-key lock 也会被锁上。

假如，数据库表中当前有以下记录：

![图片](lock.assets/640-16317539699728)

当我们执行`select id from t where c=5 lock in share mode`的时候：

- 根据原则 1，加锁单位是 next-key lock，因此会给 (0,5]加上 next-key lock。要注意 c 是普通索引，因此仅访问 c=5 这一条记录是不能马上停下来的，需要向右遍历，查到 c=10 才放弃。
- 根据原则 2，访问到的都要加锁，因此要给 (5,10]加 next-key lock。
- 根据优化 2：等值判断，向右遍历，最后一个值不满足 c=5 这个等值条件，因此退化成间隙锁 (5,10)。
- 根据原则 2 ，只有访问到的对象才会加锁，这个查询使用覆盖索引，并不需要访问主键索引，所以主键索引上没有加任何锁。

当我们执行`select * from t where c>=10 and c<11 for update`的时候：

- 根据原则 1，加锁单位是 next-key lock，会给 (5,10]加上 next-key lock，范围查找就往后继续找，找到 id=15 这一行停下来
- 根据原则 2，访问到的都要加锁，因此需要加 next-key lock(10,15]。
- 由于索引 c 是非唯一索引，没有优化规则，也就是说不会蜕变为行锁，因此最终 sesion A 加的锁是，索引 c 上的 (5,10] 和 (10,15] 这两个 next-key lock。

### **总结**

以上，我们介绍了InnoDB中的锁机制，一共有三种锁，分别是Record Lock、Gap Lock和Next-Key Lock。

Record Lock表示记录锁，锁的是索引记录。Gap Lock是间隙锁，说的是索引记录之间的间隙。Next-Key Lock是Record Lock和Gap Lock的组合，同时锁索引记录和间隙。他的范围是左开右闭的。

InnoDB的RR级别中，加锁的基本单位是 next-key lock，只要扫描到的数据都会加锁。唯一索引上的范围查询会访问到不满足条件的第一个值为止。

同时，为了提升性能和并发度，也有两个优化点：

- 索引上的等值查询，给唯一索引加锁的时候，next-key lock 退化为行锁。
- 索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，next-key lock 退化为间隙锁。

关于锁的介绍，就是这么多了，但是其实，RR的隔离级别引入的这些锁，虽然一定程度上可解决很多如幻读这样的问题，但是也会带来一些副作用，比如并发度降低、容易导致死锁等。

后面我们再来单独介绍一下为什么RR作为InnoDB的默认级别，却"不受待见"，很多大厂都会把数据库默认级别修改为RC。