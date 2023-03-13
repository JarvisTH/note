**一、引言**

*1.桥梁模式用意*

将抽象化与实现化脱耦，使得二者可以独立变化。

*抽象化*：存在于多个实体中的共同的概念性关系。通常，一组对象如果具有相同概念性联系，可以通过一个共同类描述，复杂一点的可以用抽象类和具体子类描述。

*实现化*：抽象化给出的具体实现。

*脱耦*：将两个实体的行为的某种强关联去掉。这里指将抽象化与实现化之间的耦合改为弱关联。

强关联是在编译时期确定的，无法在运行时期动态改变。弱关联是可以动态的确定并且可以在运行时期动态改变的关联。继承是强关联，聚合是弱关联。

**二、桥梁模式结构**
![](https://upload-images.jianshu.io/upload_images/9449419-a76582c9550e5ec3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
角色：
- 抽象化角色：抽象化给出的定义，并保存一个对实现抽象化对象的引用。
- 修正抽象化角色（Refined Abstraction）：扩展抽象化角色，改变和修正父类对抽象化的定义。
- 实现化角色（Implementor）：给出实现化角色的接口，但不给出具体实现。接口不一定和抽象化角色接口定义相同。
- 具体实现化角色：给出实现化角色接口的具体实现。
```
abstract public class Abstraction{
    protected Implementor imp;

    public void operation(){
        imp.operationImp();
    }
}
```
```
public class RefinedAbstraction extends Abstraction{
    public void operation(){
      ...
    }
}
```
```
abstract public class Implementor{
    public abstract void operationImp();
}
```
```
public class ConcreteImplementorA extends Implementor{
    public void operationImp(){
        ...
    }
}
```
一般，实现化角色中的每个方法都有一个抽象化角色中的某个方法与之相对应，反过来不一定。

**三、桥梁模式实现**

*1.实现化角色退化*
在只有一个具体实现化角色情况下，抽象的实现化角色可以去掉。
![](https://upload-images.jianshu.io/upload_images/9449419-afbf9ab5f93c2ea0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*2.抽象化角色的行为*

在很多情况下，Abstraction与RefinedAbstraction并没有区别。

*3.多个实现类情况*

当有多于一个实现类时，应该在什么时间、什么地方、怎么创建一个实现类的实例？

如果抽象化角色知道具体实现角色的所有信息，那么可以在构造子里根据传进的参数决定创建哪一个具体实现化角色类实例。

另一个采用方法是，先选用一个具体实现化角色类进行实例化，然后根据情况改为另一个具体实现层类的实例。

*4.共享具体实现化角色*

可以有几个抽象化角色类合用相同的具体实现化角色类。

**四、在什么情况下使用桥梁模式**

- 如果一个系统需要在构件的抽象化角色和具体化角色间增加更多灵活性，避免在两个层次间建立静态联系。
- 设计要求实现化角色的任何改变不影响客户端，或者说实现化角色的改变对客户端透明。
- 一个构件有多于一个的抽象化角色和实现化角色，系统需要他们之间进行动态耦合。
- 设计要求独立管理抽象化角色和具体化角色的变化。

**五、从重构角度考察**

所谓重构，就是在不增加或减少一个软件系统的功能情况下，对代码结构进行优化。

*问题*
![](https://upload-images.jianshu.io/upload_images/9449419-ec0fb06eaa86b9e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
方案一：
![](https://upload-images.jianshu.io/upload_images/9449419-dc57d1f16f9519f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
缺陷：在具体飞机与飞机制造商、飞机种类之间耦合过强，出现以下情况时，系统设计需要修改：
- 需要向系统引进新的飞机制造商
- 需要向系统引进新的飞机类型

