简单工厂模式是创建模式，又叫静态工厂方法模式。简单工厂模式是由一个工厂对象决定创建出哪一种产品类的实例。

适用场景：工厂类负责创建的对象比较少。应用层对如何创建对象不关心，只知道传入工厂类的参数。

Calendar类

**一.工厂模式的几种形态**

工厂模式负责将大量有共同接口的类实例化。工厂模式可以动态决定实例化某一类。工厂模式形态有：
- 简单工厂模式
- 工厂方法（Factory Method）模式：又称多态性工厂模式或虚拟构造子模式。
- 抽象工厂模式（Abstract Factory）：又称工具箱模式。

**二、简单工厂模式的引进**

例子：
有一个农场，里面有各种水果，这个系统里需要描述的水果有：葡萄（Grape）、草莓（Strawberry）、苹果（Apple）。

一个自然的做法是建立一个各种水果都适用的接口，如图：
![](https://upload-images.jianshu.io/upload_images/9449419-5eb7779093ba6716.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
水果接口规定出所有水果必须实现的接口，包括任何水果类必须具备的方法：种植plant（），生长grow（），收获harvest（）。

水果接口代码：
```
public interface Fruit{
    //生长
    void grow();
    //收获
    void harvest();
    //种植
    void plant();
}
```
苹果是水果的一种，实现了水果的所有方法，又因为苹果是多年生植物，因此多一个treeAge属性，描述树龄。代码：
```
public class Apple implements Fruit{
     private int treeAge;
      //生长
      public void grow（）{
          log（“Apple is growing”）；
       }
      //收获
       public void harvest（）{
          log（“Apple is harvested”）；
       }
      //种植
      public void plant（）{
          log（“Apple is planted”）；
       }
      //辅助方法
      public static void log（String msg）{
          System.out.println(msg)；
       }
      //树龄取值方法
      public int getTreeAge(){
           return treeAge;
      }
      //树龄赋值方法
      public void setTreeAge（int treeAge）{
           this.treeAge=treeAge;
      }
}
```
同样，Grape是水果一种，由于分有籽和无籽两种，所有多一个seedless属性。代码：
```
public class Grape implements Fruit{
     private boolean seedless;
      //生长
      public void grow（）{
          log（“Grape is growing”）；
       }
      //收获
       public void harvest（）{
          log（“Grape is harvested”）；
       }
      //种植
      public void plant（）{
          log（“Grape is planted”）；
       }
      //辅助方法
      public static void log（String msg）{
          System.out.println(msg)；
       }
      //有籽取值方法
      public int getSeedless(){
           return seedless;
      }
      //有籽赋值方法
      public void setSeedless（boolean seedless）{
           this.seedless=seedless;
      }
}
```
Strawberry代码：
```
public class Strawberry implements Fruit{
      //生长
      public void grow（）{
          log（“Strawberry is growing”）；
       }
      //收获
       public void harvest（）{
          log（“Strawberry is harvested”）；
       }
      //种植
      public void plant（）{
          log（“Strawberry is planted”）；
       }
      //辅助方法
      public static void log（String msg）{
          System.out.println(msg)；
       }
}
```
园丁也是系统一部分，FruitGardener的结构：
![](https://upload-images.jianshu.io/upload_images/9449419-53db7dc931355bd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
该类会根据客户端要求，创建不同的水果对象，如果接到不合法要求，抛出BadFruitException异常。
![](https://upload-images.jianshu.io/upload_images/9449419-c0e495b087ddf425.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
FruitGardener类代码：
```
public class FruitGardener{
    //静态工厂方法
    public static Fruit factory（String which）throws BadFruitException{
         if（which.equalsIgnoreCase("apple"){
            return new Apple();
          }
          else if( which.equalsIgnoreCase("grape"){
            return new Grape();
          }
          else if(which.equalsIgnoreCase("strawberry"){
            return new Strawberry();
          }
          else{
              throw new BadFruitException("Bad fruit request");
           }
      }
}
```
异常类：
```
public class BadFruitException extends Exception{
    public BadFruitException (String msg){
        super(msg);
      }
}
```
使用时，客户端只需要调用FruitGardener的静态方发factory即可。
```
try{
    FruitGardener.factory("apple");
    ...
  }catch(BadFruitException e){
      ..
}
```

**三、简单工厂模式的结构**

简单工厂模式是由工程类根据传入的参数决定创建哪一种产品类的实例。
![](https://upload-images.jianshu.io/upload_images/9449419-e8ab18db730475e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
简单工厂模式涉及三个角色：
- 工厂类角色（Creator）
- 抽象产品（Product）角色：担任由工厂类所创建产品的父类或共同拥有的接口
- 具体产品（Concrete Product）角色

抽象产品角色主要是给所有具体产品类提供一个共同类型，最简单时可以简化为标识接口。

**四、简单工厂模式的实现**
1.多层次产品
![](https://upload-images.jianshu.io/upload_images/9449419-77bea7b9c67e700e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
好处是设计简单，产品类的等级结构不会反应到工厂类中，产品类的等级结构变化不会影响工厂类。缺点是增加新的产品将导致工厂类修改。

2.使用Java接口或抽象类

3.多个工厂方法

多个工厂方法分别复杂创建不同产品对象。

4.抽象产品角色省略

如果系统仅有一个具体产品角色，可以省略掉抽象产品角色。
![](https://upload-images.jianshu.io/upload_images/9449419-9ebebe5667c01a3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5.工厂角色与抽象产品角色合并
![](https://upload-images.jianshu.io/upload_images/9449419-22e6daedfa2442e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6.三个角色全部合并
如果抽象产品角色已被省略，而工厂角色可以与具体产品角色合并。
![](https://upload-images.jianshu.io/upload_images/9449419-8a48e8d81c471617.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/9449419-944ab03c51ac3c16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
7.产品对象的循环使用和登记式工厂方法

工厂方法可以通过登记它所创建的产品对象来达到循环使用产品对象的目的。例如单例模式与多例模式。

**五、简单工厂模式优缺点**
优点是使得客户端免除直接创建产品对象的责任，而仅仅复杂消费产品。

缺点是系统扩展较为困难，工厂角色无法形成基于继承的等级结构。有限的支持开闭原则，扩展时无需修改消费角色，但要修改工厂角色。

