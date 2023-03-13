属于结构模式，又叫部分-整体模式。合成模式将对象组织到树结构中，可以用来描述整体与部分关系。

**一、对象的树结构**

有三种树结构：从上到下，从下到上，双向的。

**二、安全式和透明式的合成模式**
![](https://upload-images.jianshu.io/upload_images/9449419-adf091bac3008b4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
角色：
- 抽象构件角色：给参与组合的对象规定一个接口。这个对象给出共有的接口及其默认行为。
- 树叶构件角色：代表参与组合的树叶对象。
- 树枝构件角色：代表参加组合的有子对象的对象，并给出树枝结构对象的行为。

合成描述根据实现接口区别分为两种形式，分别是安全式和透明式。

*1.透明方式*

在Component里面声明所有的用来管理子类对象的方法，这样做的好处是所有构件类有相同的接口。在客户端看来，树叶类对象与合成类对象的区别起码在接口层次上消失了，客户端同等对待所有对象。

缺点是不够安全，因为树叶类对象和合成类对象在本质上有区别，树叶类对象不可能有下一层，因此add，remove，getChild方法没有意义，运行时可能出错。

*2.安全方式*

在Composite类里面声明所有用来管理子类对象的方法。这样做安全是因为树叶类型对象没有管理子类对象的方法，因此客户端无法使用这些管理方法。

缺点是不够透明，因为树叶类和合成类将具有不同的接口。

**三、安全式的合成模式结构**
![](https://upload-images.jianshu.io/upload_images/9449419-5a5ab4d0d8368dbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
角色：
- 抽象构件角色：给参加组合的对象定义出公共的接口及其默认行为，可以用来管理所有子对象。合成对象通常把它所包含的子对象当作类型为Composite的对象。
- 树叶构件角色：给出参加组合的原始对象的行为。
- 树枝构件角色：代表参加组合的有下级子对象的对象，给出所有管理子对象的方法。
```
public interface Component{
    Composite getComposite（）；
    
    void sampleOperation（）；
}
```
```
public class Composite implements Component{
    private Vector componentVector=new java.util.Vector();
    
    public Composite getComposite(){
    return this;
    }

    public void sampleOperation(){
        Enumeration enmueration=components();
        while(enumeration.hasMoreElements()){
            ((Component)enumeration.nextElement()).sampleOperation();
        }
    }

    public void add(Component component){
    componentVector.addElement(component);
    }

    public void remove(Component component){
    componentVector.removeElement(component);
    }

    public Enumeration components(){
    return component Vector.elements();
    }
 }
```
```
public class Leaf implements Component{
    public void sample Operation（）{
     ...
    }

  public Composite getComposite(){
        return null;
    }
}
```

**四、透明式的合成模式结构**
![](https://upload-images.jianshu.io/upload_images/9449419-d85dfb9fb4d0867a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
角色：
- 抽象构件角色：给参与的对象规定一个接口，规范共有的接口及默认行为。
- 树叶构件角色：代表参与组合的树叶对象，定义出参加组合的原始对象行为。
- 树枝构件角色：代表参加组合的有子对象的对象，定义出这样的对象的行为。
```
public interface Component{
    void sampleOperation();

    Composite getComposite();

    void add(Component component);
  
    void remove(Conponent component);

    Enumeration components();
}
```
```
public class Composite implements Component{
    private Vector componentVector=new Vector();

    public Composite getComposite(){
        return this;
    }

    public void sampleOperation(){
        Enumeration enumeration=components();
        while(enumeration.hasMoreElements()){
            ((Component)enumeration.nextElement()).sampleOperation();
          }
    }

     public void add(Component component){
         componentVector.addElement(component);
    }

    public void remove(Component component){
          componentVector.removeElement(component);
    }

    public Enumeration components(){
        return componentVector.elements();
    }
}
```
```
public class Leaf implements Component{
    public void sampleOperation(){
        ...
    }

     public void add(Component component){
    }

      public void remove(Component component){
    }

     public Composite getComposite(){
        return null;
    }

    public Enumeration components(){
        return null;
    }
}
```
**五、合成模式的实现**

实现时考虑：
- 明显的给出父对象的引用。在子对象里给出父对象的引用，可以容易的遍历所有父对象，管理合成结构。定义出这个父对象引用恰当的地方是Composite角色。

一般一个对象持有所有子对象引用，没有父对象引用。当子对象持有父对象引用时，构成由下向上的树结构，反之，构成由上向下结构。既持有父引用，又持有所有子对象引用，则是双向的树结构。

- 在通常的系统，可以使用享元模式实现构件共享，但是由于合成模式的对象经常要有对父类的引用，因此不容易实现共享。
- 抽象构件类(Component）应当多“重”才好。一种思路是应当尽量”重“，目的是使客户端不知道它所使用的是哪一个特殊的树叶构件或树枝构件，意味着所有具体构件类都有相同接口。这时，有些方法只适用于树叶构件，另一些适用于树枝构件。

另一种思路是使Component对象较“轻”，因为管理子对象的部分接口不适用于树叶，所以避免把合成部件的特有接口移到抽象部件中。即前面讨论的安全式与透明式。

- 有时候系统需要遍历过一个树枝构件的子构件很多次，这时可以把遍历子构件的结构暂时放在父构件中，作为缓存。

- 使用什么数据类型来存储子对象。

- Composite向子类的委派。客户端不应当直接调用树叶类，应当由其父类向树叶类进行委派。增加代码复用。

