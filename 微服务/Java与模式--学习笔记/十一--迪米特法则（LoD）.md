又叫最少知道原则，一个对象对其他对象的了解应当尽可能少。若两个类不必直接通信，那么这两个类就不应该发生直接相互作用。

**一、从代码重构角度理解**
可以利用转发方法的形式解决。
![](https://upload-images.jianshu.io/upload_images/9449419-0813fdd87f8e7318.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上图中，Friend持有一个Stranger对象的引用，说明Friend类知道Stranger类。下面是someone代码：
```
public class Someone{
  public void opt1(Friend friend){
        Stranger stranger=friend.provide();
         stranger.opt3();
      }
  }
  ```
Someone具有方法opt1（），参数是Friend类型，所以Someone知道Friend，而Friend的provide方法提供自己所创建的Stranger实例。
显然，Someone的方法opt1（）不满足LoD，因为这个方法引用了Stranger对象，而Someone不应该知道Stranger。

改造方法就是调用转发。
![](https://upload-images.jianshu.io/upload_images/9449419-c2a7279bc3b63bd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ```
public class Someone{
  public void opt1(Friend friend){
        friend.forward();
      }
}
```
```
public class Friend{
      public Stranger stranger=new Stranger();
      public void opt2(){
            ...
      }
      public void forward(){
            stranger.opt3();
      }
}
 ```
由于转发调用，使得调用的具体细节被隐藏在Friend内部，从而省略了Someone和Stranger的联系，降低了系统耦合度。修改时，仅仅直接影响这个类知道的类，而不会影响它不知道的类。

缺点：会在系统里造出大量的小方法，散落在系统各处。使得系统不同模块之间通信效率降低，不同模块之间不容易协调。

为了克服缺点，可以使用依赖倒转原则。

**二、广义的LoD**
一个设计的好模块可以将他所有的实现细节隐藏起来，彻底的将提供给外界的API和自己的实现分隔。
信息隐藏的重要性原因在于可以使得各个子系统间脱耦，从而运行他们独立的被开发、优化、使用、修改等。

运用LoD设计时，要注意：
- 在类划分上，应当创建有弱耦合的类。类间耦合越弱，越有利于复用。
- 在类的结构设计上，每个类都应该尽量降低成员的访问权限。
- 在类的设计上，只要有可能，一个类应当设计成不变的类。
- 在对其他类的引用上，一个对象对其他对象的引用应当降到最低。

强调只和朋友交流，不和陌生人说话，朋友指的出现在成员变量、方法输入、输出参数中的类称为成员朋友类，而出现在方法体内部的类不属于朋友类。
