https://mp.weixin.qq.com/s/b38zctK3Y0JeboitLyTp4A

# 盘点 HashMap 源码中的那些优雅的设计！

## **一、HashMap构造器**

HashMap总共给我们提供了三个构造器来创建HashMap对象。

1.无参构造函数`public HashMap()`：

使用无参构造函数创建的hashmap对象，其默认容量为16，默认的负载因子为0.75。

2.有参构造函数`public HashMap(int initialCapacity，float loadFactor)`：

使用该构造函数，我们可以指定hashmap的初始化容量和负载因子，但是在hashmap底层不一定会初始化成我们传入的容量，而是会初始化成大于等于传入值的最小的2的幂次方，比如我们传入的是17，那么hashmap会初始化成`32（2^5）`。那么hashmap是如何高效计算**大于等于一个数的最小2的幂次方**数的呢，源码如下：

```
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
  }
```

它的设计可以说很巧妙，其基本思想是如果一个二进制数低位全是1，那么这个数+1则肯定是一个2的幂次方数。举个例子看一下：

![图片](HashMap 源码中的那些优雅的设计.assets/1)

可以看到，它的计算过程是：首先将我们指定的那个数cap减1（减1的原因是，如果cap正好是一个2的幂次方数，也可以正确计算），然后对cap-1分别无符号右移1位、2位，4位、8位、16位(加起来正好是31位)，并且每次移位后都与上一个数做按位或运算，通过这样的运算，会使得最终的结果低位都是1。那么最终对结果加1，就会得到一个2的幂次方数。

3.另一个有参构造函数就是有参构造函数`public HashMap(int initialCapacity)`,

该构造函数和上一个构造函数唯一不同之处就是不能指定负载因子

## **二、HashMap插入机制**

#### 1.插入方法源码

```java
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 初始化桶数组 table， table 被延迟到插入新数据时再进行初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 如果桶中不包含键值对节点引用，说明当前数组下标下不存在任何数据，则将新键值对节点的引用存入桶中即可
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            //如果hash相等，并且equals方法返回true，这说明key相同，此时直接替换value即可，并且返回原值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
             //如果第一个节点是树节点，则调用putTreeVal方法，将当前值放入红黑树中
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
               //如果第一个节点不是树节点，则说明还是链表节点，则开始遍历链表，将值存储到链表合适的位置
                for (int binCount = 0; ; ++binCount) {
                     //如果遍历到了链接末尾，则创建链表节点，将数据存储到链表结尾
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //判断链表中节点树是否超多了阈值8，如果超过了则将链表转换为红黑树（当然不一定会转换，treeifyBin方法中还有判断）
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果在链表中找到，完全相同的key，则直接替换value
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //e!=null说明只是遍历到中间就break了，该种情况就是在链表中找到了完全相等的key,该if块中就是对value的替换操作
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //加入value之后，更新size，如果超过阈值，则进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

![图片](HashMap 源码中的那些优雅的设计.assets/2)

（1）在put一个k-v时，首先调用hash()方法来计算key的hashcode,而在hashmap中并不是简单的调用key的hashcode求出一个哈希码，还用到了扰动函数来降低哈希冲突。源码如下：

```java
static final int hash(Object key) {
     int h;
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
 }
```

从源码中可以看到，最终的哈希值是将原哈希码和原哈希码右移16位得到的值进行异或运算的结果。16正好是32的一半，因此hashmap是将hashcode的高位移动到了低位，再通过异或运算将高位散播的低位，从而降低哈希冲突。

至于为什么能够降低冲突呢，我们可以看看作者对hash方法的注释：

> ```
> /**
>  * Computes key.hashCode() and spreads (XORs) higher bits of hash
>  * to lower.  Because the table uses power-of-two masking, sets of
>  * hashes that vary only in bits above the current mask will
>  * always collide. (Among known examples are sets of Float keys
>  * holding consecutive whole numbers in small tables.)  So we
>  * apply a transform that spreads the impact of higher bits
>  * downward. There is a tradeoff between speed, utility, and
>  * quality of bit-spreading. Because many common sets of hashes
>  * are already reasonably distributed (so don't benefit from
>  * spreading), and because we use trees to handle large sets of
>  * collisions in bins, we just XOR some shifted bits in the
>  * cheapest possible way to reduce systematic lossage, as well as
>  * to incorporate impact of the highest bits that would otherwise
>  * never be used in index calculations because of table bounds.
>  */
> ```

> 从注释中我们可以得出，作者进行高位向低位散播的原因是：由于hashmap在计算bucket下标时，计算方法为hash&n-1,n是一个2的幂次方数，因此hash&n-1正好取出了hash的低位，比如n是16，那么hash&n-1取出的是hash的低四位，那么如果多个hash的低四位正好完全相等，这就导致了always collide（冲突），即使hash不同。因此将高位向低位散播，让高位也参与到计算中，从而降低冲突，让数据存储的更加散列。

（2）在计算出hash之后之后，调用putVal方法进行key-value的存储操作。在putVal方法中首先需要判断table是否被初始化了（因为hashmap是延迟初始化的，并不会在创建对象的时候初始化table），如果table还没有初始化，则通过resize方法进行扩容。

```java
if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
```

（3）通过(n-1)&hash计算出当前key所在的bucket下标，如果当前table中当前下标中还没有存储数据，则创建一个链表节点直接将当前k-v存储在该下标的位置。

```java
if ((p = tab[i = (n - 1) & hash]) == null)
     tab[i] = newNode(hash, key, value, null);
```

（4）如果table下标处已经存在数据，则首先判断当前key是否和下标处存储的key完全相等，如果相等则直接替换value，并将原有value返回，否则继续遍历链表或者存储到红黑树。

```java
if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))
      e = p;
```

（5）当前下标处的节点是树节点，则直接存储到红黑树中

```
else if (p instanceof TreeNode)
         e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
```

（6）如果不是红黑树，则遍历链表，如果在遍历链表的过程中，找到相等的key，则替换value，如果没有相等的key，就将节点存储到链表尾部（jdk8中采用的是尾插法），并检查当前链表中的节点树是否超过了阈值8，如果超过了8，则通过调用treeifyBin方法将链表转化为红黑树。

```java
for (int binCount = 0; ; ++binCount) {
      if ((e = p.next) == null) {
          p.next = newNode(hash, key, value, null);
          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
              treeifyBin(tab, hash);
          break;
      }
      if (e.hash == hash &&
          ((k = e.key) == key || (key != null && key.equals(k))))
          break;
      p = e;
  }
```

（7）将数据存储完成之后，需要判断当前hashmap的大小是否超过扩容阈值`Cap*load_fact`,如果大于阈值，则调用`resize()`方法进行扩容。

```java
f (++size > threshold)
       resize();
```



HashMap在扩容后的容量为原容量的2倍，起基本机制是创建一个2倍容量的table，然后将数据转存到新的散列表中，并返回新的散列表。和jdk1.7中不同的是，jdk1.8中多转存进行了优化，可以不再需要重新计算bucket下标，其实现源码如下：

![图片](HashMap 源码中的那些优雅的设计.assets/3)

从源码中我们可以看出，如果一个key hash和原容量oldCap按位与运算结果为0，则扩容前的bucket下标和扩容后的bucket下标相等，否则扩容后的bucket下标是原下标加上oldCap。

使用的基本原理总结如下：

> 1、如果一个数m和一个2的幂次方数n进行按位与运算不等于0，则有：`m&(n2-1)=m&(n-1)+n`理解：一个2的幂次方数n，在二进制中只有一位为1（假设第k位是1），其他位均为0，那个如果一个数m和n进行按位与运算结果为0的话，则说明m的二进制第k位肯定为0，那么m的前n位和前n-1位所表示的值肯定是相等的。

> 2、如果一个数m和一个2的幂次方数n进行按位与运算等于0，则有：`m&(n2-1)=m&(n-1)`理解：一个2的幂次方数n，在二进制中只有一位为1（假设第k位是1），其他位均为0，那个如果一个数m和n进行按位与运算结果不为0的话，则说明m的二进制第k位肯定为1，那么m的前n位和前n-1位所表示的值的差恰好是第k位上的1所表示的数，二这个数正好是n。

![图片](HashMap 源码中的那些优雅的设计.assets/4)

```
Java网站推荐：www.java1000.com，网站包括Java基础、进阶、源码、面试等各个系列文章，欢迎浏览！Github仓库推荐：https://github.com/OUYANGSIHAI/JavaInterview，复制链接直达，该仓库是本人面试一年的面试记录与分享，相信对你有一定的帮助！
```