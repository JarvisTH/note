**一、引言**

属于结构模式，以共享的方式高效的持有大量的细粒度对象。提供了减少对象数量从而改善应用的对象结构的方式。

适用场景：
- 常应用于系统底层开发，以便解决系统的性能问题
- 系统有大量相似对象、需要缓冲池场景

优点：
- 降低对象的创建，降低内存中对象的数量，降低系统内存
- 减少内存之外其他资源占用

缺点：
- 关注内/外部状态、关注线程安全问题
- 使系统程序的逻辑复杂化

扩展：
- 内部状态
- 外部状态

内部状态是存储在享元对象内部的，并且不会随环境改变，因此一个享元可以具有内蕴状态并共享。

外部状态是随环境变化的、不可以共享的状态。享元对象的外蕴状态必须由客户端保存，并在享元对象被创建后，在需要使用时传入享元对象内部。

相关设计模式：
- 和代理模式
- 和单例模式

源码中的应用：
jdk的integer.valueOf等方法

*1.种类*

分为单纯享元模式和复合享元模式。

**二、单纯享元模式结构**
![](https://upload-images.jianshu.io/upload_images/9449419-f8561b6c4ea23c15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
角色：
- 抽象享元模式：所有具体享元类的超类，为这些类规定出需要实现的公共接口。
- 具体享元角色：实现抽象享元角色所规定的接口。如果有内蕴状态，必须负责为内蕴状态提供存储空间。
- 享元工厂角色：负责创建和管理享元角色，必须保证享元对象可以被系统适当共享。当客户端调用一个享元对象时，享元工厂会检查系统中是否已经有一个符合要求的享元对象，如果有，则享元工厂提供这个享元对象；如果没有一个适当的享元对象，工厂角色就创建一个。
- 客户端角色：需要维护一个对所有享元对象的引用。存储所有享元角色的外蕴状态。
```
abstract public class Flyweight{
    //state为外蕴状态
    abstract public void operation(String state);
}
```
```
public class ConcreteFlyweight extends Flyweight{
    private Character intrinsicState=null;
    
    //内蕴状态作为参数传入
    public ConcreteFlyweight(Character state){
        this.intrinsicState=state;
    }

    public void operation(String state){
        .......//不改变内蕴状态
    }
}
```
客户端不可以直接将具体享元类实例化，必须通过工厂对象，利用factory方法得到享元对象，一般享元工厂可以使用单例模式。
```
public class FlyweightFactory{
    private HashMap flies=new HashMap();
    
    private Flyweight lnkFlyweight;

    public FlyweightFactory(){}

    public Flyweight factory(Charactory state){
        if(flies.containsKey(state)){
            return (Flyweight)flies.get(state);
        }else{
            Flyweight fly=new ConcreteFlyweight(state);
            flies.put(state,fly);
            return fly;
        }
    }

    public void checkFlyweight(){
        Flyweight fly;
        int i=0;
        for(Iterator it=flies.entrySet().iterator();it.hasNext();  ){
            Map.Entry e=(Map.Entry)it.next();
            System.out.println(...);
        }
    }
}
```
调用：
```
FlyweightFactory facotry=new FlyweightFactory();

Flyweight fly=factory.factory(new Character('a'));

 fly.operation("first call");

 fly=factory.facory(new Character('b'));

 fly.operation("second call");

fly=factory.facory(new Character('a'));

 fly.operation("third call");
```

**三、复合享元模式结构**
![](https://upload-images.jianshu.io/upload_images/9449419-d7fe9baf24fa94b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
复合享元模式本身不能共享，但可以分解为单纯享元对象。

角色：
- 抽象享元角色：是所有具体享元类的超类，为这些类规定出需要实现的公共接口。抽象享元的接口使得享元变得可能，但不强制子类共享，因此并非所有享元对象都可以共享。
- 具体享元角色：实现抽象享元角色所规定的接口。
如果有内蕴状态，必须负责为内蕴状态提供存储空间。
- 复合享元角色：复合享元角色所代表的对象是不可以共享的，但一个复合享元对象可以分解为多个本身是单纯享元对象的组合。
- 享元工厂角色：负责创建和管理享元角色，保证享元对象可以被系统适当共享。
- 客户端角色
```
abstract public class Flyweight{
    //state为外蕴状态
    abstract public void operation(String state);
}
```
```
public class ConcreteFlyweight extends Flyweight{
    private Character intrinsicState=null;
    
    //内蕴状态作为参数传入
    public ConcreteFlyweight(Character state){
        this.intrinsicState=state;
    }

    public void operation(String state){
        .......//不改变内蕴状态
    }
}
```
```
public class ConcreteCompositeFlyweight{
    private HashMap flies=mew HashMap(10);
    
    private Flyweight flyweight;

    public ConcreteCompositeFlyweight(){}

    //新增一个单纯享元对象到聚集中
    public void add(Character key,Flyweight fly){
        flies.put(key,fly);
    }

    public void operation(String extrinsicState){
        Flyweight fly=null;
        for(Iterator it=flies.entrySet().iterator();it.hasNext();  ){
            Map.Entry e=(Map.Entry)it.next();
            fly=(flyweight)e.getValue();
            fly.operation(extrinsicState);
        }
    }
}
```
```
public class FlyweightFactory{
    private HashMap flies=new HashMap();
    
    public FlyweightFactory(){}

    //复合享元工厂
    public Flyweight factory(String compositeFly){
        ConcreteCompositeFlyweight compositeFly=new ConcreteCompositeFlyweight();
        int length=compositeState.length();
        Character state=null;
        for(int i=0;i<lengthli++){
            state=new Character(compositeState.charAt(i)）；
            compositeFly.add(state,this.factory(state));
        }
        return compositeFly;
    }

    //单纯享元工厂
    public Flyweight factory(Character state){
        if(flies.containsKey(state)){
        return (Flyweight)flies.get(state);
    }else{
         Flyweight fly=new ConcreteFlyweight(state);
            flies.put(state,fly);
            return fly;
        }
    }

public void checkFlyweight(){
        Flyweight fly;
        int i=0;
        for(Iterator it=flies.entrySet().iterator();it.hasNext();  ){
            Map.Entry e=(Map.Entry)it.next();
            System.out.println(...);
        }
    }
}
```
**四、模式实现**

*1.使用不变模式实现享元角色*

*2.使用备忘录模式实现享元工厂角色*

*3.使用单例模式实现享元工厂角色*

**五、享元模式在什么情况下使用**

- 一个系统有大量对象
- 对象耗费大量内存
- 对象的状态中的大部分都可以外部化
- 对象可以按照内蕴状态分成很多组，当把外蕴对象从对象中剔除时，每个组都可以仅有一个对象代替。
- 对象可以是不可分辨的。
