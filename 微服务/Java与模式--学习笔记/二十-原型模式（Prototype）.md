通过给出一个原型对象来指明所要创建的对象的类型，然后复用这个原型对象的办法创建更多同类型的对象。（原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象）

特点：不需要指导任何细节，不调用任何构造函数。

适用场景：
- 类初始化消耗资源较多
- new产生一个对象需要非常繁琐过程（数据准备、访问权限等）
- 构造函数复杂
- 循环体中产生大量对象时

优点：
- 性能比直接new对象性能高
- 创建简单

缺点：
- 必须配备克隆方法
- 对克隆复杂对象或对克隆出的对象进行复杂改造时，容易引入风险
- 深拷贝、浅拷贝要运用得当

原型模式的克隆破坏单例。

jdk中的应用：
实现了Cloneable的类；

**一、引言**

所有类继承自java.lang.Object，而该类通过clone方法，要使用该方法必须实现标识接口Cloneable。

**二、Java对象的复制**

*1.满足克隆的条件*

- 克隆对象与原对象不是同一个对象
- 克隆对象与原对象类型一样
- 如果对象x的equals方法定义恰当，那么x.clone().equals(x)成立。

*2.equals（）方法*

```
public boolean equals(Object obj){
    return (this==obj);
}
```
当两个变量指向同一对象时,equals（）才会返还true。

**三、原型模式结构**

*1.简单形式*
![](https://upload-images.jianshu.io/upload_images/9449419-f0ce3f591eaf9165.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
角色：
- 客户角色：提出创建对象请求
- 抽象原型角色：通常由接口或者抽象类实现，给出具体原型类所需接口。
- 具体原型角色：被复制的对象，需要实现抽象的原型角色要求的接口。
```
public class Client{
    private Prototype prototype;
  
    public void operation(Prototype example){
        Prototype p=(Prototype) example.clone();
    }
}
```
```
public interface Prototype extends Cloneable{
    Prototype clone();
}
```
```
public class ConcretePrototype implements Prototype{
    public Object clone(){
        try{
            return super.clone();
        }catch(CloneNotSupportedException e){
            return null;
        }
    }
}
```

*2.登记式*

![](https://upload-images.jianshu.io/upload_images/9449419-2bea1298741d608c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
角色：
客户端角色：向管理员提出创建对象请求
- 抽象原型角色：通常由接口或者抽象类实现，给出具体原型类所需接口。
- 具体原型角色：被复制的对象，需要实现抽象的原型角色要求的接口。
- 原型管理器角色：创建具体原型类的对象，并记录每个被创建的对象。

Prototype的源代码同上。

```
public class ConcretePrototype implements Prototype{
 
    public synchronized Object clone(){
        Prototype temp=null;
        try{
            temp=(Prototype)super.clone();
            return temp;
        }catch(CloneNotSupportedException e){
            ...
        }finally{
            return temp;
    }
}
```
```
public class PrototypeManager{
    private Vector objects=new Vector();

    public void add(Prototype object){
        objects.add(object);
    }

    public Prototype get(int i){
        return (Prototype)objects.get(i);
    }

    public int getSize(){
        return objects.size();
    }
}
```
```
public class Client{
    private PrototypeManager mgr;
    private Prototype prototype;
    public void registerPrototype(){
    prototype=new ConcretePrototype();
    Prototype copytype=(Prototype)prototype.clone();
    mgr.add(copytype);
    }
}
```
如果需要创建的原型对象数目较少可以使用第一种，数目不固定可以采取第二种。

**四、深复制和浅复制**

*1.浅复制*

浅复制仅仅复制所考虑的对象，而不复制它所引用的对象。

*2.深复制*

深复制要把复制的对象所引用的对象都要复制一遍。

*3.利用串行化做深复制*

把对象写到流的过程叫串行化过程。写入到流的是对象的一个拷贝，而原对象仍然存在JVM中。

在Java中深复制一个对象，常常可以使对象实现Serializable接口，然后把对象写到一个流里，再从流读出来，便可以重建一个对象。
```
public Object deepClone(){
    //写入流
    ByteArrayOutputStream bo=new ByteArrayOutputStream();
    ObjectOutputStream oo=new  ObjectOutputStream(bo);
    oo.writeObject(this);
    //从流读出
    ByteArrayInputStream bi=new ByteArrayInputStream(bo.toByteArray());
    ObjectInputStream oi=new ObjectInputStream(bi);
    return (oi.readObject());
}
```
前提是对象以及对象内部所有引用的对象都可以串行化，否则需要考虑那些不可串行化的对象可否设为transient，排除复制过程。
对于类似线程对象或Socket对象，不能简单复制或者共享，应该设置为transient。

**五、什么情况下使用原型模式

对于产品结构经常变化的系统，采用工厂模式不方便，采用原型模式，给每个产品类配备一个克隆方法，便可以避免使用工厂模式带来的具有固定等级结构的工厂类。

**六、原型模式的优缺点**

优点：
- 允许动态的增加或减少产品类。
- 提供简化的创建结构。
- 具有给一个应用软件动态加载新功能的能力。
- 原型模式适用于任何等级结构。

缺点是每个类必须配备一个克隆方法。


