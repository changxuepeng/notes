https://blog.csdn.net/qq_29411737/article/details/80835658

# 1.Lambda表达式

> lambda表达式本质上是一段匿名内部类，也可以是一段可以传递的代码

先来体验一下lambda最直观的优点：简洁代码

```java
//匿名内部类
  Comparator<Integer> cpt = new Comparator<Integer>() {
      @Override
      public int compare(Integer o1, Integer o2) {
          return Integer.compare(o1,o2);
      }
  };

  TreeSet<Integer> set = new TreeSet<>(cpt);

  System.out.println("=========================");

  //使用lambda表达式
  Comparator<Integer> cpt2 = (x,y) -> Integer.compare(x,y);
  TreeSet<Integer> set2 = new TreeSet<>(cpt2);
```

这样一个场景，在商城浏览商品信息时，经常会有条件的进行筛选浏览，例如要选颜色为红色的、价格小于8000千的….

## 优化一：使用设计模式

定义一个MyPredicate接口

```java
public interface MyPredicate <T> {
    boolean test(T t);
}
```

如果想要筛选颜色为红色的商品，定义一个颜色过滤类

```java
public class ColorPredicate implements MyPredicate <Product> {

     private static final String RED = "红色";

     @Override
     public boolean test(Product product) {
         return RED.equals(product.getColor());
     }
}
```

定义过滤方法，将过滤接口当做参数传入，这样这个过滤方法就不用修改，在实际调用的时候将具体的实现类传入即可。

```java
public List<Product> filterProductByPredicate(List<Product> list,MyPredicate<Product> mp){
        List<Product> prods = new ArrayList<>();
        for (Product prod : list){
            if (mp.test(prod)){
                prods.add(prod);
            }
        }
        return prods;
    }

```

例如，如果想要筛选价格小于8000的商品，那么新建一个价格过滤类既可

```java
public class PricePredicate implements MyPredicate<Product> {
    @Override
    public boolean test(Product product) {
        return product.getPrice() < 8000;
    }
}
```

## 优化二：使用匿名内部类

定义过滤方法：

```java
public List<Product> filterProductByPredicate(List<Product> list,MyPredicate<Product> mp){
        List<Product> prods = new ArrayList<>();
        for (Product prod : list){
            if (mp.test(prod)){
                prods.add(prod);
            }
        }
        return prods;
    }
```

调用过滤方法的时候：

```java
// 按价格过滤
public void test2(){
    filterProductByPredicate(proList, new MyPredicate<Product>() {
        @Override
        public boolean test(Product product) {
            return product.getPrice() < 8000;
        }
    });
}

 // 按颜色过滤
 public void test3(){
     filterProductByPredicate(proList, new MyPredicate<Product>() {
         @Override
         public boolean test(Product product) {
             return "红色".equals(product.getColor());
         }
     });
 }
```

使用匿名内部类，就不需要每次都新建一个实现类，直接在方法内部实现。看到匿名内部类，不禁想起了Lambda表达式。

## 优化三：使用lambda表达式

定义过滤方法：

```java
public List<Product> filterProductByPredicate(List<Product> list,MyPredicate<Product> mp){
        List<Product> prods = new ArrayList<>();
        for (Product prod : list){
            if (mp.test(prod)){
                prods.add(prod);
            }
        }
        return prods;
    }
```

使用lambda表达式进行过滤

```java
@Test
public void test4(){
      List<Product> products = filterProductByPredicate(proList, (p) -> p.getPrice() < 8000);
      for (Product pro : products){
          System.out.println(pro);
      }
  }
```

## 优化四：使用Stream API

甚至不用定义过滤方法，直接在集合上进行操作

```java
// 使用jdk1.8中的Stream API进行集合的操作
@Test
public void test(){
    // 根据价格过滤
    proList.stream()
           .fliter((p) -> p.getPrice() <8000)
           .limit(2)
           .forEach(System.out::println);

    // 根据颜色过滤
    proList.stream()
           .fliter((p) -> "红色".equals(p.getColor()))
           .forEach(System.out::println);

    // 遍历输出商品名称
    proList.stream()
           .map(Product::getName)
           .forEach(System.out::println);
}
```

Lmabda表达式的语法总结： () -> ();

|                  **前置**                  |                       **语法**                       |
| :----------------------------------------: | :--------------------------------------------------: |
|               无参数无返回值               |       () -> System.out.println(“Hello WOrld”)        |
|             有一个参数无返回值             |             (x) -> System.out.println(x)             |
|          有且只有一个参数无返回值          |              x -> System.out.println(x)              |
|  有多个参数，有返回值，有多条lambda体语句  | (x，y) -> {System.out.println(“xxx”);return xxxx;}； |
| 有多个参数，有返回值，只有一条lambda体语句 |                    (x，y) -> xxxx                    |

口诀：左右遇一省括号，左侧推断类型省

注：当一个接口中存在多个抽象方法时，如果使用lambda表达式，并不能智能匹配对应的抽象方法，因此引入了函数式接口的概念

# 2.函数式接口

> 函数式接口的提出是为了给Lambda表达式的使用提供更好的支持。

什么是函数式接口？
简单来说就是只定义了一个抽象方法的接口（Object类的public方法除外），就是函数式接口，并且还提供了注解：@FunctionalInterface

常见的四大函数式接口

### 1.Consumer 《T》：消费型接口，有参无返回值

```java
  @Test
    public void test(){
        changeStr("hello",(str) -> System.out.println(str));
    }

    /**
     *  Consumer<T> 消费型接口
     * @param str
     * @param con
     */
    public void changeStr(String str, Consumer<String> con){
        con.accept(str);
    }
```

### 2.Supplier 《T》：供给型接口，无参有返回值

```java
 @Test
    public void test2(){
        String value = getValue(() -> "hello");
        System.out.println(value);
    }

    /**
     *  Supplier<T> 供给型接口
     * @param sup
     * @return
     */
    public String getValue(Supplier<String> sup){
        return sup.get();
    }
```

### 3.Function 《T,R》：:函数式接口，有参有返回值

```java
  @Test
    public void test3(){
        Long result = changeNum(100L, (x) -> x + 200L);
        System.out.println(result);
    }

    /**
     *  Function<T,R> 函数式接口
     * @param num
     * @param fun
     * @return
     */
    public Long changeNum(Long num, Function<Long, Long> fun){
        return fun.apply(num);
    }
```

### 4.Predicate《T》： 断言型接口，有参有返回值，返回值是boolean类型

```java
public void test4(){
        boolean result = changeBoolean("hello", (str) -> str.length() > 5);
        System.out.println(result);
    }

    /**
     *  Predicate<T> 断言型接口
     * @param str
     * @param pre
     * @return
     */
    public boolean changeBoolean(String str, Predicate<String> pre){
        return pre.test(str);
    }
```

在四大核心函数式接口基础上，还提供了诸如BiFunction、BinaryOperation、toIntFunction等扩展的函数式接口，都是在这四种函数式接口上扩展而来的，不做赘述。

总结：函数式接口的提出是为了让我们更加方便的使用lambda表达式，不需要自己再手动创建一个函数式接口，直接拿来用就好了


# 3.方法引用

> 若lambda体中的内容有方法已经实现了，那么可以使用“方法引用”
> 也可以理解为方法引用是lambda表达式的另外一种表现形式并且其语法比lambda表达式更加简单

### (a) 方法引用

三种表现形式：

1. 对象：：实例方法名
2. 类：：静态方法名
3. 类：：实例方法名 （lambda参数列表中第一个参数是实例方法的调用 者，第二个参数是实例方法的参数时可用）

```java
public void test() {
        /**
        *注意：
        *   1.lambda体中调用方法的参数列表与返回值类型，要与函数式接口中抽象方法的函数列表和返回值类型保持一致！
        *   2.若lambda参数列表中的第一个参数是实例方法的调用者，而第二个参数是实例方法的参数时，可以使用ClassName::method
        *
        */
        Consumer<Integer> con = (x) -> System.out.println(x);
        con.accept(100);

        // 方法引用-对象::实例方法
        Consumer<Integer> con2 = System.out::println;
        con2.accept(200);

        // 方法引用-类名::静态方法名
        BiFunction<Integer, Integer, Integer> biFun = (x, y) -> Integer.compare(x, y);
        BiFunction<Integer, Integer, Integer> biFun2 = Integer::compare;
        Integer result = biFun2.apply(100, 200);

        // 方法引用-类名::实例方法名
        BiFunction<String, String, Boolean> fun1 = (str1, str2) -> str1.equals(str2);
        BiFunction<String, String, Boolean> fun2 = String::equals;
        Boolean result2 = fun2.apply("hello", "world");
        System.out.println(result2);
    }
```

### (b)构造器引用

格式：ClassName::new

```java
public void test2() {

        // 构造方法引用  类名::new
        Supplier<Employee> sup = () -> new Employee();
        System.out.println(sup.get());
        Supplier<Employee> sup2 = Employee::new;
        System.out.println(sup2.get());

        // 构造方法引用 类名::new （带一个参数）
        Function<Integer, Employee> fun = (x) -> new Employee(x);
        Function<Integer, Employee> fun2 = Employee::new;
        System.out.println(fun2.apply(100));
 }
```

### (c)数组引用

格式：Type[]::new

```java
public void test(){
        // 数组引用
        Function<Integer, String[]> fun = (x) -> new String[x];
        Function<Integer, String[]> fun2 = String[]::new;
        String[] strArray = fun2.apply(10);
        Arrays.stream(strArray).forEach(System.out::println);
}
```

# 4.Stream API

Stream操作的三个步骤

- 创建stream
- 中间操作（过滤、map）
- 终止操作

### 1.stream的创建：

```java
 // 1，校验通过Collection 系列集合提供的stream()或者paralleStream()
    List<String> list = new ArrayList<>();
    Strean<String> stream1 = list.stream();

    // 2.通过Arrays的静态方法stream()获取数组流
    String[] str = new String[10];
    Stream<String> stream2 = Arrays.stream(str);

    // 3.通过Stream类中的静态方法of
    Stream<String> stream3 = Stream.of("aa","bb","cc");

    // 4.创建无限流
    // 迭代
    Stream<Integer> stream4 = Stream.iterate(0,(x) -> x+2);

    //生成
    Stream.generate(() ->Math.random());
```

### 2.Stream的中间操作:

```java
/**
   * 筛选 过滤  去重
   */
  emps.stream()
          .filter(e -> e.getAge() > 10)
          .limit(4)
          .skip(4)
          // 需要流中的元素重写hashCode和equals方法
          .distinct()
          .forEach(System.out::println);


  /**
   *  生成新的流 通过map映射
   */
  emps.stream()
          .map((e) -> e.getAge())
          .forEach(System.out::println);


  /**
   *  自然排序  定制排序
   */
  emps.stream()
          .sorted((e1 ,e2) -> {
              if (e1.getAge().equals(e2.getAge())){
                  return e1.getName().compareTo(e2.getName());
              } else{
                  return e1.getAge().compareTo(e2.getAge());
              }
          })
          .forEach(System.out::println);
```

### 3.Stream的终止操作：

```java
 /**
         *      查找和匹配
         *          allMatch-检查是否匹配所有元素
         *          anyMatch-检查是否至少匹配一个元素
         *          noneMatch-检查是否没有匹配所有元素
         *          findFirst-返回第一个元素
         *          findAny-返回当前流中的任意元素
         *          count-返回流中元素的总个数
         *          max-返回流中最大值
         *          min-返回流中最小值
         */

        /**
         *  检查是否匹配元素
         */
        boolean b1 = emps.stream()
                .allMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
        System.out.println(b1);

        boolean b2 = emps.stream()
                .anyMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
        System.out.println(b2);

        boolean b3 = emps.stream()
                .noneMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
        System.out.println(b3);

        Optional<Employee> opt = emps.stream()
                .findFirst();
        System.out.println(opt.get());

        // 并行流
        Optional<Employee> opt2 = emps.parallelStream()
                .findAny();
        System.out.println(opt2.get());

        long count = emps.stream()
                .count();
        System.out.println(count);

        Optional<Employee> max = emps.stream()
                .max((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
        System.out.println(max.get());

        Optional<Employee> min = emps.stream()
                .min((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));
        System.out.println(min.get());
```

还有功能比较强大的两个终止操作 reduce和collect
reduce操作： reduce:(T identity,BinaryOperator)/reduce(BinaryOperator)-可以将流中元素反复结合起来，得到一个值

```java
    /**
         *  reduce ：规约操作
         */
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
        Integer count2 = list.stream()
                .reduce(0, (x, y) -> x + y);
        System.out.println(count2);

        Optional<Double> sum = emps.stream()
                .map(Employee::getSalary)
                .reduce(Double::sum);
        System.out.println(sum);
```

collect操作：Collect-将流转换为其他形式，接收一个Collection接口的实现，用于给Stream中元素做汇总的方法

```java
  /**
         *  collect：收集操作
         */

        List<Integer> ageList = emps.stream()
                .map(Employee::getAge)
                .collect(Collectors.toList());
        ageList.stream().forEach(System.out::println);
```

# 5.Optional容器

> 使用Optional容器可以快速的定位NPE，并且在一定程度上可以减少对参数非空检验的代码量。

```java
/**
     *      Optional.of(T t); // 创建一个Optional实例
     *      Optional.empty(); // 创建一个空的Optional实例
     *      Optional.ofNullable(T t); // 若T不为null，创建一个Optional实例，否则创建一个空实例
     *      isPresent();    // 判断是够包含值
     *      orElse(T t);   //如果调用对象包含值，返回该值，否则返回T
     *      orElseGet(Supplier s);  // 如果调用对象包含值，返回该值，否则返回s中获取的值
     *      map(Function f): // 如果有值对其处理，并返回处理后的Optional，否则返回Optional.empty();
     *      flatMap(Function mapper);// 与map类似。返回值是Optional
     *
     *      总结：Optional.of(null)  会直接报NPE
     */

Optional<Employee> op = Optional.of(new Employee("zhansan", 11, 12.32, Employee.Status.BUSY));
        System.out.println(op.get());

        // NPE
        Optional<Employee> op2 = Optional.of(null);
        System.out.println(op2);
@Test
    public void test2(){
        Optional<Object> op = Optional.empty();
        System.out.println(op);

        // No value present
        System.out.println(op.get());
    }
@Test
    public void test3(){
        Optional<Employee> op = Optional.ofNullable(new Employee("lisi", 33, 131.42, Employee.Status.FREE));
        System.out.println(op.get());

        Optional<Object> op2 = Optional.ofNullable(null);
        System.out.println(op2);
       // System.out.println(op2.get());
    }
    @Test
    public void test5(){
        Optional<Employee> op1 = Optional.ofNullable(new Employee("张三", 11, 11.33, Employee.Status.VOCATION));
        System.out.println(op1.orElse(new Employee()));
        System.out.println(op1.orElse(null));
    }

    @Test
    public void test6(){
        Optional<Employee> op1 = Optional.of(new Employee("田七", 11, 12.31, Employee.Status.BUSY));
        op1 = Optional.empty();
        Employee employee = op1.orElseGet(() -> new Employee());
        System.out.println(employee);
    }

    @Test
    public void test7(){
        Optional<Employee> op1 = Optional.of(new Employee("田七", 11, 12.31, Employee.Status.BUSY));
        System.out.println(op1.map( (e) -> e.getSalary()).get());
    }
```

# 6.接口中可以定义默认实现方法和静态方法

在接口中可以使用default和static关键字来修饰接口中定义的普通方法

```java
public interface Interface {
    default  String getName(){
        return "zhangsan";
    }

    static String getName2(){
        return "zhangsan";
    }
}
```

# 7.新的日期API LocalDate | LocalTime | LocalDateTime

> 新的日期API都是不可变的，更使用于多线程的使用环境中
>
> 表示日期的LocalDate
> 表示时间的LocalTime
> 表示日期时间的LocalDateTime

新的日期API的几个优点：

>  之前使用的java.util.Date月份从0开始，我们一般会+1使用，很不方便，java.time.LocalDate月份和星期都改成了enum
>  * java.util.Date和SimpleDateFormat都不是线程安全的，而LocalDate和LocalTime和最基本的String一样，是不变类型，不但线程安全，而且不能修改。
>  * java.util.Date是一个“万能接口”，它包含日期、时间，还有毫秒数，更加明确需求取舍
>  * 新接口更好用的原因是考虑到了日期时间的操作，经常发生往前推或往后推几天的情况。用java.util.Date配合Calendar要写好多代码，而且一般的开发人员还不一定能写对
>



```java
@Test
    public void test(){
        // 从默认时区的系统时钟获取当前的日期时间。不用考虑时区差
        LocalDateTime date = LocalDateTime.now();
        //2018-07-15T14:22:39.759
        System.out.println(date);

        System.out.println(date.getYear());
        System.out.println(date.getMonthValue());
        System.out.println(date.getDayOfMonth());
        System.out.println(date.getHour());
        System.out.println(date.getMinute());
        System.out.println(date.getSecond());
        System.out.println(date.getNano());

        // 手动创建一个LocalDateTime实例
        LocalDateTime date2 = LocalDateTime.of(2017, 12, 17, 9, 31, 31, 31);
        System.out.println(date2);
        // 进行加操作，得到新的日期实例
        LocalDateTime date3 = date2.plusDays(12);
        System.out.println(date3);
        // 进行减操作，得到新的日期实例
        LocalDateTime date4 = date3.minusYears(2);
        System.out.println(date4);
    }
```

```java
 @Test
    public void test2(){
        // 时间戳  1970年1月1日00：00：00 到某一个时间点的毫秒值
        // 默认获取UTC时区
        Instant ins = Instant.now();
        System.out.println(ins);

        System.out.println(LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli());
        System.out.println(System.currentTimeMillis());

        System.out.println(Instant.now().toEpochMilli());
        System.out.println(Instant.now().atOffset(ZoneOffset.ofHours(8)).toInstant().toEpochMilli());
    }
```

```java
@Test
    public void test3(){
        // Duration:计算两个时间之间的间隔
        // Period：计算两个日期之间的间隔

        Instant ins1 = Instant.now();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Instant ins2 = Instant.now();
        Duration dura = Duration.between(ins1, ins2);
        System.out.println(dura);
        System.out.println(dura.toMillis());

        System.out.println("======================");
        LocalTime localTime = LocalTime.now();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        LocalTime localTime2 = LocalTime.now();
        Duration du2 = Duration.between(localTime, localTime2);
        System.out.println(du2);
        System.out.println(du2.toMillis());
    }
```

```java
@Test
    public void test4(){
        LocalDate localDate =LocalDate.now();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        LocalDate localDate2 = LocalDate.of(2016,12,12);
        Period pe = Period.between(localDate, localDate2);
        System.out.println(pe);
    }
```

```java
 @Test
    public void test5(){
        // temperalAdjust 时间校验器
        // 例如获取下周日  下一个工作日
        LocalDateTime ldt1 = LocalDateTime.now();
        System.out.println(ldt1);

        // 获取一年中的第一天
        LocalDateTime ldt2 = ldt1.withDayOfYear(1);
        System.out.println(ldt2);
        // 获取一个月中的第一天
        LocalDateTime ldt3 = ldt1.withDayOfMonth(1);
        System.out.println(ldt3);

        LocalDateTime ldt4 = ldt1.with(TemporalAdjusters.next(DayOfWeek.FRIDAY));
        System.out.println(ldt4);

        // 获取下一个工作日
        LocalDateTime ldt5 = ldt1.with((t) -> {
            LocalDateTime ldt6 = (LocalDateTime)t;
            DayOfWeek dayOfWeek = ldt6.getDayOfWeek();
            if (DayOfWeek.FRIDAY.equals(dayOfWeek)){
                return ldt6.plusDays(3);
            }
            else if (DayOfWeek.SATURDAY.equals(dayOfWeek)){
                return ldt6.plusDays(2);
            }
            else {
                return ldt6.plusDays(1);
            }
        });
        System.out.println(ldt5);
    }
```

```java
 @Test
    public void test6(){
        // DateTimeFormatter: 格式化时间/日期
        // 自定义格式
        LocalDateTime ldt = LocalDateTime.now();
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日");
        String strDate1 = ldt.format(formatter);
        String strDate = formatter.format(ldt);
        System.out.println(strDate);
        System.out.println(strDate1);

        // 使用api提供的格式
        DateTimeFormatter dtf = DateTimeFormatter.ISO_DATE;
        LocalDateTime ldt2 = LocalDateTime.now();
        String strDate3 = dtf.format(ldt2);
        System.out.println(strDate3);

        // 解析字符串to时间
        DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        LocalDateTime time = LocalDateTime.now();
        String localTime = df.format(time);
        LocalDateTime ldt4 = LocalDateTime.parse("2017-09-28 17:07:05",df);
        System.out.println("LocalDateTime转成String类型的时间："+localTime);
        System.out.println("String类型的时间转成LocalDateTime："+ldt4);
    }
```

```java
  // ZoneTime  ZoneDate       ZoneDateTime
    @Test
    public void test7(){
        LocalDateTime now = LocalDateTime.now(ZoneId.of("Asia/Shanghai"));
        System.out.println(now);

        LocalDateTime now2 = LocalDateTime.now();
        ZonedDateTime zdt = now2.atZone(ZoneId.of("Asia/Shanghai"));
        System.out.println(zdt);

        Set<String> set = ZoneId.getAvailableZoneIds();
        set.stream().forEach(System.out::println);
    }

```

### 1.LocalDate

```java
public static void localDateTest() {

        //获取当前日期,只含年月日 固定格式 yyyy-MM-dd    2018-05-04
        LocalDate today = LocalDate.now();

        // 根据年月日取日期，5月就是5，
        LocalDate oldDate = LocalDate.of(2018, 5, 1);

        // 根据字符串取：默认格式yyyy-MM-dd，02不能写成2
        LocalDate yesteday = LocalDate.parse("2018-05-03");

        // 如果不是闰年 传入29号也会报错
        LocalDate.parse("2018-02-29");
    }

```

### 2.LocalDate常用转化

```java
  /**
     * 日期转换常用,第一天或者最后一天...
     */
    public static void localDateTransferTest(){
        //2018-05-04
        LocalDate today = LocalDate.now();
        // 取本月第1天： 2018-05-01
        LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth());
        // 取本月第2天：2018-05-02
        LocalDate secondDayOfThisMonth = today.withDayOfMonth(2);
        // 取本月最后一天，再也不用计算是28，29，30还是31： 2018-05-31
        LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth());
        // 取下一天：2018-06-01
        LocalDate firstDayOf2015 = lastDayOfThisMonth.plusDays(1);
        // 取2018年10月第一个周三 so easy?：  2018-10-03
        LocalDate thirdMondayOf2018 = LocalDate.parse("2018-10-01").with(TemporalAdjusters.firstInMonth(DayOfWeek.WEDNESDAY));
    }

```

### 3.LocalTime

```java
 public static void localTimeTest(){
        //16:25:46.448(纳秒值)
        LocalTime todayTimeWithMillisTime = LocalTime.now();
        //16:28:48 不带纳秒值
        LocalTime todayTimeWithNoMillisTime = LocalTime.now().withNano(0);
        LocalTime time1 = LocalTime.parse("23:59:59");
    }
```

### 4.LocalDateTime

```java
public static void localDateTimeTest(){
        //转化为时间戳  毫秒值
        long time1 = LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli();
        long time2 = System.currentTimeMillis();

        //时间戳转化为localdatetime
        DateTimeFormatter df= DateTimeFormatter.ofPattern("YYYY-MM-dd HH:mm:ss.SSS");

        System.out.println(df.format(LocalDateTime.ofInstant(Instant.ofEpochMilli(time1),ZoneId.of("Asia/Shanghai"))));
    }

```

