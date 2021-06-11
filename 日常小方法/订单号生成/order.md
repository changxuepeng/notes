
https://mp.weixin.qq.com/s?__biz=MzAxNjk4ODE4OQ==&mid=2247492664&idx=2&sn=bb0bdf960fe9402a18621c832083bc37



去年年底的时候，我们线上出了一次事故，这个事故的表象是这样的:

> 系统出现了两个一模一样的订单号，订单的内容却不是不一样的，而且系统在按照 订单号查询的时候一直抛错，也没法正常回调，而且事情发生的不止一次，所以 这次系统升级一定要解决掉。

经手的同事之前也改过几次，不过效果始终不好：总会出现订单号重复的问题， 所以趁着这次问题我好好的理了一下我同事写的代码。

这里简要展示下当时的代码：

```java
 /**
    * OD单号生成
    * 订单号生成规则：OD + yyMMddHHmmssSSS + 5位数(商户ID3位+随机数2位) 22位
    */
   public static String getYYMMDDHHNumber(String merchId){
          StringBuffer orderNo = new StringBuffer(new SimpleDateFormat("yyMMddHHmmssSSS").format(new Date()));
          if(StringUtils.isNotBlank(merchId)){
              if(merchId.length()>3){
                  orderNo.append(merchId.substring(0,3));
              }else {
                  orderNo.append(merchId);
              }
          }
          int orderLength = orderNo.toString().length();
          String randomNum = getRandomByLength(20-orderLength);
          orderNo.append(randomNum);
          return orderNo.toString();
   }
  
  
      /** 生成指定位数的随机数 **/
      public static String getRandomByLength(int size){
          if(size>8 || size<1){
              return "";
          }
          Random ne = new Random();
          StringBuffer endNumStr = new StringBuffer("1");
          StringBuffer staNumStr = new StringBuffer("9");
          for(int i=1;i<size;i++){
              endNumStr.append("0");
              staNumStr.append("0");
          }
          int randomNum = ne.nextInt(Integer.valueOf(staNumStr.toString()))+Integer.valueOf(endNumStr.toString());
          return String.valueOf(randomNum);
      }
```

可以看到，这段代码写的其实不怎么好，代码部分暂且不议，代码中使订单号不重复的主要因素点是随机数和毫秒，可是这里的随机数只有两位

在高并发环境下极容易出现重复问题，同时毫秒这一选择也不是很好，在多核CPU多线程下，一定时间内(极小的)这个毫秒可以说是固定不变的(测试验证过)，所以这里我先以100个并发测试下这个订单号生成，测试代码如下：

```java
 public static void main(String[] args) {
        final String merchId = "12334";
        List<String> orderNos = Collections.synchronizedList(new ArrayList<String>());
        IntStream.range(0,100).parallel().forEach(i->{
            orderNos.add(getYYMMDDHHNumber(merchId));
        });

        List<String> filterOrderNos = orderNos.stream().distinct().collect(Collectors.toList());

        System.out.println("生成订单数："+orderNos.size());
        System.out.println("过滤重复后订单数："+filterOrderNos.size());
        System.out.println("重复订单数："+(orderNos.size()-filterOrderNos.size()));
    }
```

果然，测试的结果如下：

```
生成订单数：100
过滤重复后订单数：87
重复订单数：13
```

对这一烫手的山竽拿到手里没有一个清晰的解决方案可是不行的，我大概花了6+分钟和同事商量了下业务场景，决定做如下更改：

- 去掉商户ID的传入(按同事的说法,传入商户ID也是为了防止重复订单的，事实证明并没有叼用)
- 毫秒仅保留三位(缩减长度同时保证应用切换不存在重复的可能)
- 使用线程安全的计数器做数字递增(三位数最低保证并发800不重复,代码中我给了4位)
- 更换日期转换为java8的日期类以格式化(线程安全及代码简洁性考量)

经过以上思考后我的最终代码是：

```java
  /** 订单号生成(NEW) **/
    private static final AtomicInteger SEQ = new AtomicInteger(1000);
    private static final DateTimeFormatter DF_FMT_PREFIX = DateTimeFormatter.ofPattern("yyMMddHHmmssSS");
    private static ZoneId ZONE_ID = ZoneId.of("Asia/Shanghai");
    public static String generateOrderNo(){
        LocalDateTime dataTime = LocalDateTime.now(ZONE_ID);
        if(SEQ.intValue()>9990){
            SEQ.getAndSet(1000);
        }
        return  dataTime.format(DF_FMT_PREFIX)+SEQ.getAndIncrement();
    }
```

当然代码写完成了可不能这么随随便便结束了，现在得走一个测试main函数看看：

```java
 public static void main(String[] args) {

        List<String> orderNos = Collections.synchronizedList(new ArrayList<String>());
        IntStream.range(0,8000).parallel().forEach(i->{
            orderNos.add(generateOrderNo());
        });

        List<String> filterOrderNos = orderNos.stream().distinct().collect(Collectors.toList());

        System.out.println("生成订单数："+orderNos.size());
        System.out.println("过滤重复后订单数："+filterOrderNos.size());
        System.out.println("重复订单数："+(orderNos.size()-filterOrderNos.size()));
    }
    
    /**
        测试结果： 
        生成订单数：8000
        过滤重复后订单数：8000
        重复订单数：0
    **/
```

然而，我回过头来看以上代码，虽然最大程度解决了并发单号重复的问题，不过对于我们的系统架构还是有一个潜在的隐患：如果当前应用有多个实例(集群)难道就没有重复的可能了？

鉴于此问题就必然需要一个有效的解决方案，所以这时我就思考：多个实例应用订单号如何区分开呢？以下为我思考的大致方向：

- 使用UUID(在第一次生成订单号时初始化一个)
- 使用redis记录一个增长ID
- 使用数据库表维护一个增长ID
- 应用所在的网络IP
- 应用所在的端口号
- 使用第三方算法(雪花算法等等)
- 使用进程ID(某种程度下是一个可行的方案)

在此我想了下，我们的应用是跑在docker里面，而且每个docker容器内的应用端口都一样，不过网路IP不会存在重复的问题，至于进程也有存在重复的可能，对于UUID的方式之前吃过亏，总之吧，redis或DB也算是一种比较好的方式，不过独立性较差。。。

同时还有一个因素也很重要，就是所有涉及到订单号生成的应用都是在同一台宿主机(linux实体服务器)上， 所以就目前的系统架构我选用了IP的方式。

一下是我的代码：

```java
import org.apache.commons.lang3.RandomUtils;

import java.net.InetAddress;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class OrderGen2Test {

    /** 订单号生成 **/
    private static ZoneId ZONE_ID = ZoneId.of("Asia/Shanghai");
    private static final AtomicInteger SEQ = new AtomicInteger(1000);
    private static final DateTimeFormatter DF_FMT_PREFIX = DateTimeFormatter.ofPattern("yyMMddHHmmssSS");
    public static String generateOrderNo(){
        LocalDateTime dataTime = LocalDateTime.now(ZONE_ID);
        if(SEQ.intValue()>9990){
            SEQ.getAndSet(1000);
        }
        return  dataTime.format(DF_FMT_PREFIX)+ getLocalIpSuffix()+SEQ.getAndIncrement();
    }

    private volatile static String IP_SUFFIX = null;
    private static String getLocalIpSuffix (){
        if(null != IP_SUFFIX){
            return IP_SUFFIX;
        }
        try {
            synchronized (OrderGen2Test.class){
                if(null != IP_SUFFIX){
                    return IP_SUFFIX;
                }
                InetAddress addr = InetAddress.getLocalHost();
                //  172.17.0.4  172.17.0.199 ,
                String hostAddress = addr.getHostAddress();
                if (null != hostAddress && hostAddress.length() > 4) {
                    String ipSuffix = hostAddress.trim().split("\\.")[3];
                    if (ipSuffix.length() == 2) {
                        IP_SUFFIX = ipSuffix;
                        return IP_SUFFIX;
                    }
                    ipSuffix = "0" + ipSuffix;
                    IP_SUFFIX = ipSuffix.substring(ipSuffix.length() - 2);
                    return IP_SUFFIX;
                }
                IP_SUFFIX = RandomUtils.nextInt(10, 20) + "";
                return IP_SUFFIX;
            }
        }catch (Exception e){
            System.out.println("获取IP失败:"+e.getMessage());
            IP_SUFFIX =  RandomUtils.nextInt(10,20)+"";
            return IP_SUFFIX;
        }
    }


    public static void main(String[] args) {
        List<String> orderNos = Collections.synchronizedList(new ArrayList<String>());
        IntStream.range(0,8000).parallel().forEach(i->{
            orderNos.add(generateOrderNo());
        });

        List<String> filterOrderNos = orderNos.stream().distinct().collect(Collectors.toList());

        System.out.println("订单样例："+ orderNos.get(22));
        System.out.println("生成订单数："+orderNos.size());
        System.out.println("过滤重复后订单数："+filterOrderNos.size());
        System.out.println("重复订单数："+(orderNos.size()-filterOrderNos.size()));
    }
}

/**
  订单样例：20082115575546011022
  生成订单数：8000
  过滤重复后订单数：8000
  重复订单数：0
**/
```

代码说明及几点建议

- generateOrderNo()方法内不需要加锁，因为AtomicInteger内使用的是CAS自旋转锁(保证可见性的同时也保证原子性,具体的请自行了解)
- getLocalIpSuffix()方法内不需要对不为null的逻辑加同步锁(双向校验锁，整体是一种安全的单例模式)
- 本人实现的方式并不是解决问题的唯一方式，具体解决问题需要视当前系统架构具体而论
- 任何测试都是必要的，我同事在前几次尝试解决这个问题后都没有自测，不测试有损开发专业性！