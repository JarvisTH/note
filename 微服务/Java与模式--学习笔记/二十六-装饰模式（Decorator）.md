以对客户端透明的方式扩展对象的功能，是继承关系的一个代替方案。在不改变原有对象的基础之上，将功能附加到对象上。

适用场景：
- 扩展一个类的功能或者给一个类添加附加职责
- 动态的给一个对象添加功能，这些功能可以再动态撤销

优点：
- 是继承的有力补充，比继承灵活
- 通过使用不同装饰类和装饰类的组合，可以实现不同效果
- 符合开闭原则

缺点：
- 增加程序复杂性，代码和类增多
- 多层装饰时，动态装饰时会更复杂

相关设计模式：
- 和代理模式
- 和适配器模式

源码中应用：
jdk——io包，springboot——transactionAwareCacheDecorator

**一、装饰模式结构**

![](https://upload-images.jianshu.io/upload_images/9449419-24a0493386ad41e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
角色：
- 抽象构件角色：给出一个抽象接口，以规范准备接收附加责任的对象。
- 具体构件角色：定义一个将要接收附加责任的类。
- 装饰角色：持有一个构件对象的实例，并定义一个与抽象构件接口一致的接口。
- 具体装饰角色：负责给构件对象贴上附加的责任。
```
public interface Component{
    void sampleOperation();
 }
```
```
public class Decorator implements Component{
    private Component component;

    public Decorator(Component component){
        this.component=component;
    }

    public  Decorator(){
        ..  
      }

    public void sampleOperation(){
        component.sampleOperation();
    }
}
```
```
public class ConcreteComponent implements Component{
    public ConcreteComponent(){
        ...
    }

    public void sampleOperation(){
        ...
    }
}
```
```
public class ConcreteDecorator extends Decorator{
    public void sampleOperation(){
        super.sampleOperation();
    }
}
```
装饰模式的对象图呈链状架构：
![](https://upload-images.jianshu.io/upload_images/9449419-85476aaa0b441069.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**二、装饰模式在什么情况下使用**

- 需要扩展一个类的功能，或者给一个类增加附加责任。

- 需要动态的给一个对象增加功能，这些功能可以再动态撤销。

- 需要增加由一些基本功能的排列组合而产生的非常大量的功能，从而使继承关系变得不现实。
       
**三、优缺点**

优点：
- 装饰模式可以提供比继承更多灵活性。
- 通过使用不同的具体装饰类以及这些装饰类的排列组合，可以创造很多不同行为组合。

缺点：
- 装饰模式比继承更加容易出错。
- 可以比使用继承关系需要较少数目的类，但是会产生比使用继承关系更多的对象，使得查错困难，而且对象都比较像。

**四、实现的讨论**

*1. 模式简化*

- 一个装饰类的接口必须与被装饰类的接口相容。
- 尽量保存Component作为一个“轻”类。
- 如果只有一个ConreteComponent类而没有抽象的Component类，那么Decorator类经常可以是ConcreteComponent的一个子类。![](https://upload-images.jianshu.io/upload_images/9449419-a8e8605fd088d6f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 如果只有一个ConreteDecorator类，那么没必要建立一个单独的Decorator类，可以把Decorator和ConcreteDecorator合并。
![](https://upload-images.jianshu.io/upload_images/9449419-69c70e788d0c79bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*2.透明性要求*

装饰模式对客户端的透明性要求程序不要声明一个ConreteDecorator类型的变量，而应当声明一个Component类型的变量：

Conponent c=new Component（）；

而不是：

ConcreteComponent c=new ConcreteDecorator（）；

*3.半透明装饰模式*

多数装饰模式的实现是半透明的。允许装饰模式改变接口，新增方法。意味着客户端可以说明ConcreteDecorator类型变量，调用该类才有的方法：

Object1 o=new Object1（）；
Object2 o2=new Object2（o）；
o2.methodInObject2（）；

如果客户端不调用属于装饰的方法，只调用Component的方法，那么装饰模式等同于透明。

半透明的装饰模式介于装饰模式和适配器模式之间。


