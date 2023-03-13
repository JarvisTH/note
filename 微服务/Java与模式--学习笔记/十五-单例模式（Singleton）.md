确保某个类只有一个实例，而且自行实例化并向整个系统提供这个实例。

优点：只有一个实例，减少内存开销；避免对资源的多重占用；严格控制访问。

缺点：没有接口，扩展困难；

重点：
- 私有构造器
- 线程安全
- 延迟加载
- 序列化和反序列化安全
- 反射

相关设计模式：
- 单例模式和工厂模式
- 单例模式和享元模式

破坏单例模式的方法：
- 反射
- 序列化
- 克隆


克隆破环单例模式——解决方式：
- 单例类不实现Cloneable接口
- 或者在实现的clone方法中调用getInstance方法

**一、引言**

*1.单例模式要点*

要点有三个：
- 某个类只能有一个实例
- 必须是自行创建这个实例
- 必须向整个系统提供这个实例

*2.资源管理*

一些资源管理器常常设计成单例模式。例如，windows回收站。

**二、单例模式的结构**

将单例模式推广到任意多个有限实例，这时候就称为多例模式（Multion Pattern）和多例类。

*1.饿汉式单例类*

![](https://upload-images.jianshu.io/upload_images/9449419-968cf0c2282b2616.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
图中可以看出此类自己将自己实例化。
```
public class Singleton{
    private static final Singleton SINGLETON_INSTANCE=new Singleton();
    
    private Singleton(){}

    //静态工厂方法
    public static Singleton getInstance(){
        return SINGLETON_INSTANCE;
    }
}
```
构造方法私有，避免外部利用构造方法创建多个实例。由于构造方法私有，此类不能被继承。

*2.懒汉式单例类*

与饿汉式单例类相同的是构造方法私有，不同的是懒汉式单例类在第一次被引用时才实例化自己。
![](https://upload-images.jianshu.io/upload_images/9449419-81dc80ee4b868d73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
public class LazySingleton{
    private static final LazySingleton INSTANCE=null;
    
    private LazySingleton(){}

    //静态工厂方法
    public synchronized static LazySingleton getInstance(){
        if(INSTANCE==null){
            INSTANCE=new LazySingleton();
          }
          return INSTANCE;
    }
}
```
对静态工厂方法使用了同步化，以保证多线程安全。

*3.登记式单例类*

克服了饿汉式单例类和懒汉式单例类均不可继承的缺点。
![](https://upload-images.jianshu.io/upload_images/9449419-059ea4f9c719dfcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
public class RegSingleton{
    static final HashMap registry=new HashMap();
    static{
        RegSingleton x=new RegSingleton();
        registry.put(x.getClass().getName(),x);
    }
    
    private RegSingleton(){}

    //静态工厂方法
    public static RegSingleton getInstance(String name){
        if(name==null){
            name="RegSingleton";
          }
          if(registry.get(name)==null){
          try{
              registry.put(name,Class.forName(name).newInstance();
           }catch(Exception e){
              System.out.println("Error");
           }
         }
         return (RegSingleton)registry.get(name);
    }
}
```
它的子类需要父类帮助才能实例化。
![](https://upload-images.jianshu.io/upload_images/9449419-8b71e6d77825e158.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
子类代码：
```
public class RegSingletonChild extends RegSingleton{
    public RegSingletonChild(){}

    //静态工厂方法
    public static RegSingletonChild getInstance(){
          return (RegSingletonChild)RegSingleton.getInstance("RegSingletonChild");
    }
}
```
由于子类必须允许父类以构造函数调用产生实例，因此，子类构造函数是公开的。这样产生的实例不在父类登记中，是缺点之一。必须父类实例存在才有可能有子类实例，有些情况下，浪费空间。

**三、什么情况下使用单例**

*1.使用单例模式条件*

一个必要条件是在一个系统要求一个类只有一个实例时才能使用单例模式。

**四、单例类的状态**

*1.有状态的单例类*

一个有状态的单例对象一般是可变单例对象。有状态的可变单例对象常常当作状态库使用。例如，一个单例对象给系统提供唯一序列号。也可以持有一个聚集，允许存储多个状态。

*2.没有状态的单例类*

仅用做提供工具性函数的对象。因为目的是提供工具性函数，所以没有必要创建多个实例，适合单例模式。一个没有状态的单例类是不变单例类。

*3.多个JVM系统的分散式系统*
一个J2EE应用系统可能分布在多个JVM中，这时可能造成多个单例类实例出现在不同JVM中。在任何使用了EJB、RMI、JINI技术的分散式系统中，避免使用有状态的单例模式。

*4.多个类加载器*

当两个类加载器同时加载同一个类时，会出现两个实例。在很多J2EE服务器允许同一个服务器有几个Servlet引擎，每个引擎都有独立的类加载器，经由不同类加载器加载的对象之间是绝缘的。

如果系统没有协调机制，尽量避免使用有状态的单例类。

**五、双重检查方式的论述**

有两者方法：
- 让所有线程都不进行指令重排序——volatile
- 允许一个线程重排序，不允许其他线程看到重排序——静态内部类。

在JDK1.5之前是有问题的。不能工作的基本原因在于，无序写入造成的。如果一个线程在没有同步化的条件下读取INSTANCE引用，并调用这个对象的方法，可能发现对象的初始化过程还没完成，从而造成崩溃。

JDK1.5之后，可以使用volatile关键字修饰变量来解决无序写入产生的问题，因为volatile关键字的一个重要作用是禁止指令重排序，即保证不会出现内存分配、返回对象引用、初始化这样的顺序，从而使得双重检测真正发挥作用。
```
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}
```

**六、序列化与反序列化**
![](https://upload-images.jianshu.io/upload_images/9449419-0286d3f2d3be8e93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
序列化后，再反序列化得到的不是同一个实例。

解决方法可以在单例代码中添加方法如下：
![](https://upload-images.jianshu.io/upload_images/9449419-67bbf431a92b30b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再次允许测试代码得到同一个实例：
![](https://upload-images.jianshu.io/upload_images/9449419-e0ae7c94fff94948.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
readObject方法并不是Object的方法，而是反射出来的，过程如下：
进入ois.readObject()方法，发现调用了readObject0（）方法，进入readObject0方法，选择TC_OBJECT，发现调用了checkResolve(readOrdinaryObject())两个方法，先进入内部的readOrdinaryObject（）方法，该方法内部有obj对象，存在一个判断obj =desc.isInstantiable() ? desc.newInstance() : null;进入isInstantiable（）方法，可以发现返回的是bool值，obj为null的话，进入readOrdinaryObject（）方法的判断代码，发现调用了desc.hasReadResolveMethod（）方法，进入该方法，查看注释说明：defines a conformant readResolve method.  Otherwise, returns false.。之前在单例中实现了这个方法，接下来就会被反射调用，最后返回obj对象。

**七、反射高级解决方案和原理**
![](https://upload-images.jianshu.io/upload_images/9449419-b0863b7686005d26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
接下来，通过反射开放构造函数的权限，如下：
![](https://upload-images.jianshu.io/upload_images/9449419-34f07e26c4adff05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
成功实例化对象，两个对象不同。饿汉模式是在类加载时就生成一个实例，可以在类构造器中加一个判断，进行反射防御：
![](https://upload-images.jianshu.io/upload_images/9449419-72a4a98176e67562.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再次运行结果：
![](https://upload-images.jianshu.io/upload_images/9449419-093f3c62ccebee69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种防御方法对于类加载时生成实例的单例模式有效，因此也包括静态内部类。以下是静态内部类：
![](https://upload-images.jianshu.io/upload_images/9449419-c1f031a7395693ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
添加反射防御代码后：
![](https://upload-images.jianshu.io/upload_images/9449419-53df0637569c8e95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/9449419-afb676708f9981ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**对于不是类加载时初始化实例的单例模式**：例如懒汉式。
![](https://upload-images.jianshu.io/upload_images/9449419-b61e63aa6e5f2a92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果将上面的代码加入构造函数，调换反射实例语句和单例实例语句顺序，仍然会得到两个不同的实例。**反射可以获取域的权限，也可以更改值，所以无法设置静态标志进行判断。所以，这个无法避免。**结果如下：
![](https://upload-images.jianshu.io/upload_images/9449419-d2974392daccb12d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/9449419-9383afd0dde6bb1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/9449419-4d09926988f93ab3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**八、枚举的单例模式**
```
public enum  EnumInstance {
    INSTANCE;

    private Object data;

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }

    public static EnumInstance getInstance(){
        return INSTANCE;
    }
}
```
**枚举类的特点，保证了在序列化和反序列化时不会出现多次实例化情况；在反射攻击下，也不会出现这个问题。**
![序列化与反序列化](https://upload-images.jianshu.io/upload_images/9449419-a2fa891ede1fb17d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
查看ObjectInputStream类的readObject方法，同之前的一样，调用了readObject0（）方法，进入该方法，选择TC_ENUM，调用了readEnum（）方法，进入该方法，查看注释和代码。
![反射攻击](https://upload-images.jianshu.io/upload_images/9449419-2c2eda1554dbe78e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
原因是Enum类只有一个构造器，不是无参构造器。
![](https://upload-images.jianshu.io/upload_images/9449419-d6ab27a531831998.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下面通过反射获取有参构造函数，然后传入数据，运行：
![](https://upload-images.jianshu.io/upload_images/9449419-ea997be4156494e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
仍然报错，进入newInstance代码417，有判断代码如下：
![](https://upload-images.jianshu.io/upload_images/9449419-5abf535e77b138c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
将枚举类进行反编译，反编译工具为**jad工具**，反编译结果如下：
```
// Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.
// Jad home page: http://www.kpdus.com/jad.html
// Decompiler options: packimports(3) 
// Source File Name:   EnumInstance.java

package com.jarvis.design.creational.singleton;


public final class EnumInstance extends Enum
{

    public static EnumInstance[] values()
    {
        return (EnumInstance[])$VALUES.clone();
    }

    public static EnumInstance valueOf(String name)
    {
        return (EnumInstance)Enum.valueOf(com/jarvis/design/creational/singleton/EnumInstance, name);
    }

    private EnumInstance(String s, int i)
    {
        super(s, i);
    }

    public Object getData()
    {
        return data;
    }

    public void setData(Object data)
    {
        this.data = data;
    }

    public static EnumInstance getInstance()
    {
        return INSTANCE;
    }

    public static final EnumInstance INSTANCE;
    private Object data;
    private static final EnumInstance $VALUES[];

    static 
    {
        INSTANCE = new EnumInstance("INSTANCE", 0);
        $VALUES = (new EnumInstance[] {
            INSTANCE
        });
    }
}
```
**类被final修饰，构造器私有，INSTANCE是静态的，通过静态块在类加载时初始化，所以是线程安全的。**

枚举类还可以调用枚举实例方法，如下：
![](https://upload-images.jianshu.io/upload_images/9449419-b9ea6c0d4b4b658b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/9449419-96dd36a0172089df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过jad反编译枚举代码如下：
```
// Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.
// Jad home page: http://www.kpdus.com/jad.html
// Decompiler options: packimports(3) 
// Source File Name:   EnumInstance.java

package com.jarvis.design.creational.singleton;

import java.io.PrintStream;

public abstract class EnumInstance extends Enum
{

    public static EnumInstance[] values()
    {
        return (EnumInstance[])$VALUES.clone();
    }

    public static EnumInstance valueOf(String name)
    {
        return (EnumInstance)Enum.valueOf(com/jarvis/design/creational/singleton/EnumInstance, name);
    }

    private EnumInstance(String s, int i)
    {
        super(s, i);
    }

    protected abstract void printTest();

    public Object getData()
    {
        return data;
    }

    public void setData(Object data)
    {
        this.data = data;
    }

    public static EnumInstance getInstance()
    {
        return INSTANCE;
    }


    public static final EnumInstance INSTANCE;
    private Object data;
    private static final EnumInstance $VALUES[];

    static 
    {
        INSTANCE = new EnumInstance("INSTANCE", 0) {

            protected void printTest()
            {
                System.out.println("test");
            }

        }
;
        $VALUES = (new EnumInstance[] {
            INSTANCE
        });
    }
}
```

**九、容器单例**
```
public class ContainerSingleton {
    private static Map<String,Object> singletonMap=new HashMap<String, Object>();

    public static void putInstance(String key,Object instance){
        if(StringUtils.isNotBlank(key) && instance != null){
            if(!singletonMap.containsKey(key)){
                singletonMap.put(key,instance);
            }
        }
    }

    public static Object getInstance(String key){
        return singletonMap.get(key);
    }
}
```
适合程序初始化时，将多个单例对象放入这个容器中，统一管理，使用时通过key获取对象。**在多线程中，仍然存在隐患，hashMap不是线程安全的。**不考虑序列化和反射之类的情况，可以在一定情景中使用。

