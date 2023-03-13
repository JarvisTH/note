将一个产品的内部表象与产品的生成过程分割开来，从而可以使一个建造过程生成具有不同的内部表象的产品对象。

优点:封装性好，创建与使用分离；扩展性好，建造类间独立，一定程度解耦。

缺点：会产生多余的Builder对象，产品内部变化，建造者都要修改，成本比较大。


与工厂模式区别：工厂模式注重创建产品，建造者模式注重方法调用顺序。场景对象的粒度不同，建造者模式可以创建复杂产品，工厂模式创建产品相同。

StringBuilder.append；sqlSessionFactoryBuilder;ImmutableSet.copyOf等；

**一、引言**

*1.产品内部表象**

一个产品常有不同的组成成分作为产品零件，通常叫做产品的内部表象。不同产品可以有不同内部表象。使用建造模式可以使客户端不需要知道所生成的产品对象有哪些零件等细节。

*2.对象性质的建造*

建造产品的过程是建造零件的过程。由于建造零件过程复杂，所以建造过程被外部化到另一个称为建造者的对象里，建造者对象返还给客户端的一个全部零件都建造完毕的产品对象。

**二、建造模式的结构**

*1.类图与角色**
![](https://upload-images.jianshu.io/upload_images/9449419-af3321f57f135344.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
角色：
- 抽象建造者角色：给出一个抽象接口。以规范产品对象的各个组成成分的建造。
- 具体建造者角色：实现抽象建造者所声明的接口，给出完成创建产品实例的操作。在建造过程完成后提供实例。
- 导演者角色：调用具体建造者角色以创建产品对象。
- 产品角色：建造的对象。

一般来说，每有一个产品类，就有一个对应具体建造者类。
```
public class Director{
    private Builder builder;

    public void construct(){
        builder=new ConcreteBuilder();
        builder.builderPart1();
        builder.builderPart2();
        builder.retrieveResult();
    }
}
```
```
abstract public class Builder{
    public abstract void buildPart1();

    public abstract void buildPart2();

    public abstract Product retrieveResult();
}
```
```
public class ConcreteBuilder extends Builder{
    private Product product=new Product();
  
    public Product retrieveResult(){
        return product;
    }

    public void buildPart1(){
        ...
    }

    public void buildPart2(){
        ...
    }
}
```

*3.多个产品类情况*
![](https://upload-images.jianshu.io/upload_images/9449419-dea266f0fe17a56f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
两个具体产品类提供一个共同接口，即Product。如果产品是第三方提供，不能修改，只好将retrieveProduct方法从抽象建造者角色去掉。这样具体建造者角色可以分别具有返回类型不同的retrieveProduct方法。

**三、建造模式实现**

*1.空的零件建造方法*

当一个系统有多个产品类时，怎么保证对应的具体建造者类都有同样的接口？如果有一些产品有较多的零件，另一些较少零件，建造模式还能使用吗？

产品不一定要有同样数目的零件才能使用建造模式，如果一个产品零件较少，可以使用空的零件建造方法，忽略没有的零件。

*2.省略抽象建造者角色*

如果系统只需要一个具体建造者角色，可以省略抽象建造者角色。抽象建造者角色存在的目的是规范具体建造者角色行为。

*3.省略导演者角色*

在具体建造者只有一个的情况下，如果抽象建造者角色已被省略，还可以省略导演者角色。

*4.合并建造者角色和产品角色*

在省略抽象建造者角色和导演者角色后，还可以失去具体建造者角色，使得产品自己就是自己的建造者。有时候一个产品对象有固定的几个零件，且只有这几个零件，比较适用。

*4.建造者角色可以有多个产品构造方法*

**四、什么情况下使用建造模式**

- 需要生成的产品对象有复杂的内部结构，每个内部成分可以是对象，也可以是一个对象的一个成分。
- 需要生成产品对象的属性相互依赖。建造模式可以强制实现一种分步骤进行的建造过程。有时属性并无依赖，但在产品的属性没有确定之前，产品对象不能使用。这时建造模式仍然有意义。
- 在对象建造过程中会使用系统中的其他一些对象，这些对象在产品对象的创建过程中不易得到。

建造模式效果：
- 使得产品的内部表象可以独立变化
- 每个Builder相对独立
- 最终产品易于控制






    
