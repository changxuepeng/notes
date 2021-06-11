# 策略模式

定义一系列算法，把他们一个个封装起来，并是他们可以相互替换，本模式使得算法可独立于使用它的客户而变化

#### 一、包含3中角色

> 策略(strategy)：策略是一个接口，该接口定义若干个算法标识，即定义了若干个抽象方法
>
> 具体策略(ConcreteStrategy)：具体策略是实现策略接口的类，具体策略事项策略接口所定义的抽象方法，即给出算法标识的具体算法。
>
> 上下文（context）:上下文是依赖于策略接口的类，即上下文包含有策略声明的变量。上下文中提供一个方法，该方法委托策略变量调用具体策略所事项的策略接口中的方法

ComputableStrategy.java

```java
package com.design.patterns.demo.strategyPattern.demo1;
/**
* 策略
*/
public interface ComputableStrategy {
    public double computeScore(double[] a);
}
```

StrategyOne.java

```java
package com.design.patterns.demo.strategyPattern.demo1;
/**
* 具体策略one
*/
public class StrategyOne implements ComputableStrategy {
    @Override
    public double computeScore(double[] a) {
        double score=0,sum=0;
        for (int i = 0; i <a.length ; i++) {
            sum=sum+a[i];
        }
        score = sum/a.length;
        return score;
    }
}
```

StrategyTwo.java

```java
package com.design.patterns.demo.strategyPattern.demo1;

/**
* 具体策略two
*/
public class StrategyTwo implements ComputableStrategy {
    @Override
    public double computeScore(double[] a) {
        double score=0,multi=1;
        int n = a.length;
        for (int i = 0; i <n ; i++) {
            multi = multi*a[i];
        }
        score = Math.pow(multi,1.0/n);
        return score;
    }
}
```

StrategyThree.java

```java
package com.design.patterns.demo.strategyPattern.demo1;


import java.util.Arrays;


public class StrategyThree implements ComputableStrategy {
    @Override
    public double computeScore(double[] a) {
        if(a.length <=2){
            return 0;
        }
        double score =0,sum=0;
        Arrays.sort(a);
        for (int i = 0; i <a.length-2 ; i++) {
            sum =sum +a[i];
        }
        score = sum/(a.length-2);
        return score;
    }
}
```

GymnasticsGame.java

```java
package com.design.patterns.demo.strategyPattern.demo1;
/**
* 上下文
*/
public class GymnasticsGame {
    private ComputableStrategy computableStrategy;
    public  void  setStrategy(ComputableStrategy strategy){
        this.computableStrategy=strategy;
    }
    public double getPersonScore(double [] a){
        if(a!=null){
            return computableStrategy.computeScore(a);
        }
        return  0;
    }
}
```

StrategyApplication.java 测试方法

```java
package com.design.patterns.demo.strategyPattern.demo1;
public class StrategyApplication {
    public static void main(String[] args) {
        GymnasticsGame game = new GymnasticsGame();
        game.setStrategy(new StrategyOne());
        Person zhang = new Person();
        zhang.setName("张三");
        double [] a = {9.12,9.25,8.87,9.99,6.99,7.88};
        zhang.setScore(game.getPersonScore(a));
        Person li  = new Person();
        li.setName("李四");
        double [] b ={9.15,9.26,8.97,9.89,6.97,7.89};
        li.setScore(game.getPersonScore(b));
        System.out.println("使用算数平均值方案:");
        System.out.printf("%s 最后得分：%5.3f%n",zhang.getName(),zhang.getScore());
        System.out.printf("%s 最后得分：%5.3f%n",li.getName(),li.getScore());
        game.setStrategy(new StrategyTwo());
        zhang.setScore(game.getPersonScore(a));
        li.setScore(game.getPersonScore(b));
        System.out.println("使用几何平均值方案:");
        System.out.printf("%s 最后得分：%5.3f%n",zhang.getName(),zhang.getScore());
        System.out.printf("%s 最后得分：%5.3f%n",li.getName(),li.getScore());


        game.setStrategy(new StrategyThree());
        zhang.setScore(game.getPersonScore(a));
        li.setScore(game.getPersonScore(b));
        System.out.println("使用(去掉最高、最低)算数平均值方案:");
        System.out.printf("%s 最后得分：%5.3f%n",zhang.getName(),zhang.getScore());
        System.out.printf("%s 最后得分：%5.3f%n",li.getName(),li.getScore());
    }
}
```

#### 二、策略模式的优点

上下文（Context）和具体策略(ConcreteStrategy)是松耦合关系。因此上下文只知道它奥使用某一个实现Strategy接口的实例，但 不需要知道具体是哪一个类策略模式满足‘开-闭原则’，当增加新的具体策略时，不需要修改上下文类的代码，上下文就可以引用新的具体策略的实例

#### 三、使用场景：

> 1.一个类定义了多种行为，并且这些行为在 这个类的方法中以多个条件语句的形式出现，那么可以使用策略模式避免在类中使用大量的条件语句  
>
> 2.程序不希望暴露复杂的,与算法相关的数据结构，那么可以使用策略模式封装算法 
>
>  3.需要使用一个算法的不同变体