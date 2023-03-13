属于结构模式，给一个对象提供一个代理对象，并由代理对象控制原对象的引用。代理对象在客户端和目标对象间起到中介作用。

适用场景：
- 保护目标对象
- 增强目标对象

缺点：
- 类数目增加
- 请求速度变慢
- 系统复杂度增加

扩展：
- 静态代理
- 动态代理：通过接口
- CGLib代理：通过继承

spring代理选择：
- 当Bean有实现接口时，spring会使用JDK的动态代理
- 当Bean没有实现接口时，spring会使用CGLib
- 可以强制使用CGLib
在spring配置中加入<aop:aspectj-autoproxy proxy-target-class="true"/>


代理速度对比：
- CGLib
- JDK动态代理

相关设计模式：
- 和装饰者模式：目的不同
- 和适配器模式

源码：
jdk——reflect下的proxy类等
spring——ProxyFactoryBean，
cglibAopProxy，JdkDynamicAopProxy等
mybatis——MapperProxyFactory等


**一、代理的种类**

按目的划分：
- 远程代理
- 虚拟代理：根据需要创建一个资源销毁比较大的对象，使得此对象只在需要时才会被真正创建。
- Copy-on-Write代理：虚拟代理的一种，把复制拖延到只有在客户端需要时才进行。
- 保护代理：控制对一个对象的访问，如果需要，可以给不同用户提供不同级别的使用权限。
- Cache代理：为某一个目标操作的结构提供临时储存空间，以便多个客户端共享结果。
- 防火墙代理：保护目标
- 同步化代理：使几个用户同时用一个对象而没有冲突。
- 智能引用代理：当一个对象被引用时，提供一些额外操作。

**二、代理模式结构**

角色：
- 抽象主题角色：声明了真实主题和代理主题的共同接口。
- 代理主题角色（Proxy）：代理主题角色内部有对真实主题角色相同的接口，以便可以在任何时候都可以代替真实主体；控制对真实主题的引用，负责在需要时创建真实主题对象；代理角色通常在将客户端调用传递给真实的主题之前或之后，都要执行某个操作，而不是单纯的将调用传递给真实主题对象。
- 真实主题角色：定义代理角色所代表的真实对象。
![](https://upload-images.jianshu.io/upload_images/9449419-0aa3517d3e517b2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
abstract public class Subject{
    abstract public void request();
}
```
```
public class RealSubject extends Subject{
    public RealSubject(){}

    public void request(){
        System.out.println("From real subject");
    }
}
```
```
public class ProxySubject extends Subject{
    private RealSubject realSubject;

    public ProxySubject(){}

    public void request(){
        preRequest();
        if(realSubject==null){
            realSubject=new RealSubject();
        }
        realSubject.request();
        postRequest();
    }

    private void preRequest(){
    ....
    }

    private void postRequest(){
    ...
    }
}
```
```
Subject subject=new ProxySubject();
subject.requeat();
```

**三、Java对代理模式支持**

*1.反射与动态代理*

java.lang.reflect:Proxy , InvocationHandler 和 Method。

**四、代理模式优缺点**

*1.远程代理*

可以将网络的细节隐藏，使得客户端不必考虑网络存在。
![](https://upload-images.jianshu.io/upload_images/9449419-46e20454460a4d6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

缺点是客户端可能不知道启动的是一个耗时的远程调用。

*2.虚拟代理*

优点是代理对象可以在必要时将被代理的对象加载，可以对加载过程优化。

*3.保护代理*

优点是可以在运行时间对用户的有关权限进行检查，然后在核实后决定将调用传递给被代理的对象。

*4.智能引用代理*

在访问一个对象时可以执行一些内务处理操作，比如计数。

**五、代理模式实现**

代理模式可能并不知道真正被代理的对象，而仅仅持有一个被代理对象的接口。这时代理对象不能创建被代理对象，被代理对象必须由系统其他角色创建并传入。

