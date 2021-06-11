# 装饰模式（包装器）

动态地给对象添加一些额外的职责，就功能来说,装饰模式相比生成子类更为灵活.

#### 一、装饰模式包含4中角色：

> 抽象组件（Component）：抽象组件是一个抽象类，抽象组件定义了，定义了 被装饰者 需要进行装饰的方法
>
> 具体组件(ConcreteComponent)：具体组件是抽象组件的的一个子类，具体组件的实例被称作被装饰者。
>
> 装饰(Decorator)：装饰是抽象组件的一个子类，但装饰包含了一个抽象组件声明的 变量 以保存被装饰者的引用，装饰可以是抽象类，也可以是非抽象类，如果是抽象类，那么该类的实例称作装饰者
>
> 具体装饰 （ConcreteDecorator): 是一个非抽象子类，具体装饰的实例被称作装饰者

#### 二、java.io 包中很多类是装饰角色     

Reader(抽象类，相当于抽象组件)    

 FileReader（相当于具体组件）    

 BufferedReader(相当于 装饰)

#### 三、代码

Bird.java

```java
package com.design.patterns.demo.decoratorPattern.demo1;
/**
* 抽象组件
*/
public abstract class Bird {
    // 飞翔
    public abstract int fly();
}
```

Sparrow.java

```java
package com.design.patterns.demo.decoratorPattern.demo1;
/**
* 具体组件,被装饰者
*/
public class Sparrow  extends Bird{
    public final int DISTANCE = 100;
    @Override
    public int fly() {
        return DISTANCE;
    }
}
```

Decorator.java

```java
package com.design.patterns.demo.decoratorPattern.demo1;
/**
* 装饰
*/
public abstract class Decorator extends Bird {
    protected  Bird bird;
    public Decorator(){
    }
    public Decorator(Bird bird){
        this.bird =bird;
    }
}
```

SparrowDecorator.java

```java
package com.design.patterns.demo.decoratorPattern.demo1;
/**
* 具体装饰
*/
public class SparrowDecorator extends Decorator {
    public final int DISTANCE = 50;
    public SparrowDecorator(Bird bird) {
        super(bird);
    }
    @Override
    public int fly() {
        int distance = bird.fly()+eleFlay();
        return distance;
    }
    private int eleFlay(){
        return DISTANCE;
    }
}
```

DeCoratorApplication.java

```java
package com.design.patterns.demo.decoratorPattern.demo1;
public class DeCoratorApplication {
    public void needBird(Bird bird){
        System.out.println("这只鸟能飞行"+bird.fly()+"米");
    }
    public static void main(String[] args) {
        DeCoratorApplication application = new DeCoratorApplication();
        Bird sparrow = new Sparrow();
        Bird sparrowDecorator = new SparrowDecorator(sparrow);
        Bird sparrowDecorator2 = new SparrowDecorator(sparrowDecorator);
        application.needBird(sparrow);
        application.needBird(sparrowDecorator);
        application.needBird(sparrowDecorator2);
    }
}
```

#### 四、装饰模式相对继承的优势：

> 通过继承可以改进对象的行为，对于简单的问题这样做未尝不可，但是考虑到系统扩展性，就应当注意面向对象的一个基本原则，少用继承，多用组合。就功能来说，装饰比生成子类更为灵活。

优点：

> 1.被装饰者和装饰者是松耦合的，由于装饰仅依赖于抽象组件(component),因此具体装置，只知道他是要装饰的对象是抽象组件某一个子类的实例,不需要知道是哪个具体子类
>
> 2.装饰模式满足“开-闭原则”，不必修改具体组件,就可以增加新的针对具体具体组件的具体装饰
>
> 3.可以使用多个具体装饰来装饰具体组件

使用场景：

> 程序希望动态地增强类的某个对象的功能，而不影响到该类的其他对象。     
>
> 通过继承来增强对象，不利于系统的扩展和维护