把一个类的接口换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作。

优点：
- 能提高类的透明性和复用，现有类复用不需要改变
- 目标类和适配器解耦，提高扩展性
- 符合开闭原则

缺点：
- 编写过程中需要考虑全面，可能会增加系统复杂性
- 增加代码可读难度

扩展：
- 对象适配器：组合实现
- 类适配器：继承实现

相关设计模式：
- 和外观模式

源码：
jdk-XmlAdapter等
spring-AdviceAdapter等

**一、类的适配器模式结构**
![](https://upload-images.jianshu.io/upload_images/9449419-d4775b868a974228.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
类的适配器模式把类的API转换为目标类API。

角色：
- 目标角色：期待得到的接口
- 源角色：现有需要适配的接口
- 适配器角色：把源接口转换为目标接口，这一角色不能是接口，必须是具体类。
```
public interface Target{
    //源类有的方法
    void sampleOperation1();

    //源类没有的方法
    void sampleOperation2();
}
```
```
public class Adaptee{
    public void sampleOperation1(){}
}
```
```
public class Adapter extends Adaptee implements Target{
    public void sampleOperation2(){
    ...
    }
}
```

**二、对象的适配器模式结构**

对象适配器模式不使用继承关系连接到Adaptee类，而是使用委派关系。
![](https://upload-images.jianshu.io/upload_images/9449419-698ea1c5886d19a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
角色：
- 目标角色：期待的接口，可以是具体或者抽象类。
- 源角色：需要适配的接口。
- 适配器角色：把源接口转换为目标接口。
```
public interface Target{
    //源类有的方法
    void sampleOperation1();

    //源类没有的方法
    void sampleOperation2();
}
```
```
public class Adaptee{
    public void sampleOperation1(){}
}
```
```
public class Adapter implements Target{
    private Adaptee Adaptee;
    public Adapter(Adaptee adaptee){
        super();
        this.adaptee=adaptee;
    }

    //源类有此方法，因此适配器类可以直接委派
    public void sampleOperation1(){
        adaptee.sampleOperation1();
``}

    //源类没有此方法，由适配器补充
    public void sampleOperation2(){
      ...
    }
}
```
适配器模式是将接口不同而功能相同或相近的两个接口加以转换。

对象适配器效果：
- 一个适配器可以把多种不同的源适配到同一目标。
- 与类的适配器模式比，想要置换源类的方法不容易。如果一定要置换源类方法，只好先做一个源类的子类，替换源类方法，然后把该子类当作源适配。
- 容易增加新方法。

**三、什么情况使用适配器模式**
- 不是软件设计阶段考虑的设计模式，是随着软件维护，由于不同产品、厂家造成功能类似而接口不同情况下的解决方案。
- 系统需要使用现有类，而此类接口不符合系统需要。
- 想要建立一个可以重复使用类，用于一些彼此关联不大的类，包括一些困难会将来可能引进的类一起工作。
- （对象适配器模式而言）在设计中，需要改变的多个已有的子类的接口，如果使用类的适配器模式，要针对每个子类做一个适配器类，不方便。

