# 适配器模式：

将一个类的接口转换成客户希望的列外一个接口，Addapter 模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作，

#### 一、三种角色：

> 目标(Target):目标是一个接口，该接口是客户想使用的接口。
>
> 被是配置(Adaptee):被适配者是一个已经存在的接口或者抽象类，这个接口或者抽象类需要被适配
>
> 适配器(Adapter):适配器是一个类,该类实现了目标接口并包含有被适配者的引用,即适配器的职责是对被是配置接口（或抽象类）与目标接口进行适配。

```java
package com.design.patterns.demo.adapter;
/**
* 目标 target
*/
public interface ThreeOutlet {
    public abstract void connectCurrent();
}
```

```java
package com.design.patterns.demo.adapter;
/**
* 被是配置Adaptee
*/
public interface TwoOutlet {
    public abstract  void connectCurrent();
}
```

```java
package com.design.patterns.demo.adapter;
/**
* 适配器adapter
*/
public class TreeAdapter implements  ThreeOutlet {
    private TwoOutlet outlet;
    TreeAdapter(TwoOutlet outlet){
        this.outlet = outlet;
    }
    @Override
    public void connectCurrent() {
        outlet.connectCurrent();
    }
}
```

```java
package com.design.patterns.demo.adapter;


public class Wash implements ThreeOutlet {
    private String name ;
    Wash(){
        name = "黄河洗衣机";
    }
    Wash(String name){
        this.name=name;
    }
    @Override
    public void connectCurrent() {
        turnOn();
    }


    public void turnOn(){
        System.out.println(name+"开始洗衣服");
    }
}
```

```java
package com.design.patterns.demo.adapter;


public class TV implements TwoOutlet {
    private String name;
    TV(){
        name ="长江电视机";
    }
    TV(String name ){
        this.name = name;
    }
    @Override
    public void connectCurrent() {
        trunOn();
    }
    private void trunOn(){
        System.out.println(name+"开始播放节目");
    }
}
```

```java
package com.design.patterns.demo.adapter;

public class Application1 {
    public static void main(String[] args) {
        ThreeOutlet outlet;
        Wash wash = new Wash();
        outlet = wash;
        System.out.println("使用三项插座接通电源");
        outlet.connectCurrent();
        TV tv = new TV();
        TreeAdapter adapter = new TreeAdapter(tv);
        outlet = adapter;
        System.out.println("使用两项插座接通电源");
        outlet.connectCurrent();
    }
}
```

#### 二、适配器的适配程度

> 1.完全适配:目标接口中的方法数目和被适配者接口的方法数目相等
>
> 2.不完全适配:如果目标接口中的方法数目少于 被适配者接口的方法数目， 那么适配器只能将被适配者的接口 与目标接口进行部分适配
>
> 3.剩余适配：如果目标中的方法数目大于被是配者接口的方法数目，那么适配器可将被是配者接口与目标接口 进行完全适配，但必须将目标多余的方法给出用户允许的默认实现

#### 三、适配器的优点：

> 目标与被适配者 完全解耦适配器模式满足开闭原则，当添加一个实现Adapter 接口的新的类的时候，不必修改Adapter ,Adaptee 就能拿对这个心累的实例进行适配

#### 四、适用场景

一个程序 想使用已经存在的类，但该类所实现的接口和当前程序所使用的几口不一致。
在java API 中，如果一个接口中的方法多余一个，java API 就针对该接口提供 相应的单接口适配器，如，，WindowAdapter,KeyAdapter