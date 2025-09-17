## 1.ThreadLocal简介

### 1.面试题

- ThreadLocal中ThreadLocalMap的数据结构和关系
- ThreadLocal的key是弱引用，这是为什么？
- ThreadLocal内存泄漏问题你知道吗？
- ThreadLocal中最后为什么要加remove方法？

### 2.是什么？

ThreadLocal提供**线程局部变量**。这些变量与正常的变量不同，因为每一个线程在访问ThreadLocal实例的时候（通过其get或set方法）都有自己的、独立初始化的变量副本。ThreadLocal实例通常是类中的私有静态字段，使用它的目的是希望将状态（例如，用户ID或事物ID）与线程关联起来。

### 3.能干吗？

实现**每一个线程都有自己专属的本地变量副本**（自己用自己的变量不用麻烦别人，不和其他人共享，人人有份，人各一份）。主要解决了让每个线程绑定自己的值，通过使用get()和set()方法，获取默认值或将其改为当前线程所存的副本的值从而**避免了线程安全问题**。比如8锁案例中，资源类是使用同一部手机，多个线程抢夺同一部手机，假如人手一份不是天下太平？

### 4.API介绍

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681365874862-d474b847-86c3-44b8-8456-3bdf8faffba1.png)

### 5.永远的helloworld讲起

- 问题描述：5个销售买房子，集团只关心销售总量的准确统计数，按照总销售额统计，方便集团公司给部分发送奖金--------群雄逐鹿起纷争------为了数据安全只能加锁

```java
/**
 * 需求：5个销售卖房子，集团只关心销售总量的精确统计数
 */
class House {
    int saleCount = 0;

    public synchronized void saleHouse() {
        saleCount++;
    }

}

public class ThreadLocalDemo {
    public static void main(String[] args) {
        House house = new House();
        for (int i = 1; i <= 5; i++) {
            new Thread(() -> {
                int size = new Random().nextInt(5) + 1;
                System.out.println(size);
                for (int j = 1; j <= size; j++) {
                    house.saleHouse();
                }
            }, String.valueOf(i)).start();

        }
        try {
            TimeUnit.MILLISECONDS.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "\t" + "共计卖出多少套： " + house.saleCount);
    }
}
/**
 * 3
 * 4
 * 2
 * 4
 * 2
 * main	共计卖出多少套： 15
 */
```

- 需求变更：希望各自分灶吃饭，各凭销售本事提成，按照出单数各自统计-------比如房产中介销售都有自己的销售额指标，自己专属自己的，不和别人参和。----人手一份天下安

```java
/**
 * 需求：需求变更：希望各自分灶吃饭，各凭销售本事提成，按照出单数各自统计-------比如房产中介销售都有自己的销售额指标，自己专属自己的，不和别人参和。
 */
class House {
    int saleCount = 0;

    public synchronized void saleHouse() {
        saleCount++;
    }

    ThreadLocal<Integer> saleVolume = ThreadLocal.withInitial(() -> 0);

    public void saleVolumeByThreadLocal() {
        saleVolume.set(1 + saleVolume.get());
    }


}

public class ThreadLocalDemo {
    public static void main(String[] args) {
        House house = new House();
        for (int i = 1; i <= 5; i++) {
            new Thread(() -> {
                int size = new Random().nextInt(5) + 1;
                try {
                    for (int j = 1; j <= size; j++) {
                        house.saleHouse();
                        house.saleVolumeByThreadLocal();
                    }
                    System.out.println(Thread.currentThread().getName() + "\t" + "号销售卖出：" + house.saleVolume.get());
                } finally {
                    house.saleVolume.remove();
                }
            }, String.valueOf(i)).start();

        }
        try {
            TimeUnit.MILLISECONDS.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "\t" + "共计卖出多少套： " + house.saleCount);
    }
}
/**
 * 3	号销售卖出：1
 * 4	号销售卖出：3
 * 5	号销售卖出：4
 * 2	号销售卖出：3
 * 1	号销售卖出：5
 * main	共计卖出多少套： 16
 */
```

- **线程池**中使用threadlocal，需要记得在try finally中使用**remove清除**，避免在**线程池后续复用**过程中逻辑混乱、内存泄漏

  ```java
  class MyData {
      ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);
  
      public void add() {
          threadLocal.set(1 + threadLocal.get());
      }
  }
  
  public class ThreadLocalDemo2 {
      public static void main(String[] args) {
          MyData myData = new MyData();
          ExecutorService executorService = Executors.newFixedThreadPool(3);
          try {
              for (int i = 0; i < 10; i++) {
                  executorService.submit(() -> {
                      Integer beforeInt = myData.threadLocal.get();
                      myData.add();
                      Integer afterInt = myData.threadLocal.get();
                      System.out.println(Thread.currentThread().getName() + "\t" + " beforeInt：" + beforeInt + "\t" + "afterInt: " + afterInt);
                  });
              }
          } catch (Exception e) {
              e.printStackTrace();
          } finally {
              executorService.shutdown();
          }
      }
  }
  
  pool-1-thread-3	 beforeInt：0	afterInt: 1
  pool-1-thread-2	 beforeInt：0	afterInt: 1
  pool-1-thread-1	 beforeInt：0	afterInt: 1
  pool-1-thread-3	 beforeInt：1	afterInt: 2
  pool-1-thread-3	 beforeInt：2	afterInt: 3
  pool-1-thread-1	 beforeInt：1	afterInt: 2
  pool-1-thread-3	 beforeInt：3	afterInt: 4
  pool-1-thread-1	 beforeInt：2	afterInt: 3
  pool-1-thread-3	 beforeInt：4	afterInt: 5
  pool-1-thread-1	 beforeInt：3	afterInt: 4
      
      
      try {
           Integer beforeInt = myData.threadLocal.get();
           myData.add();
           Integer afterInt = myData.threadLocal.get();
           System.out.println(Thread.currentThread().getName() + "\t" + " beforeInt：" + beforeInt + "\t" + "afterInt: " + afterInt);
      } finally {
           myData.threadLocal.remove();
      }
  pool-1-thread-3	 beforeInt：0	afterInt: 1
  pool-1-thread-2	 beforeInt：0	afterInt: 1
  pool-1-thread-1	 beforeInt：0	afterInt: 1
  pool-1-thread-3	 beforeInt：0	afterInt: 1
  pool-1-thread-1	 beforeInt：0	afterInt: 1
  pool-1-thread-3	 beforeInt：0	afterInt: 1
  pool-1-thread-1	 beforeInt：0	afterInt: 1
  pool-1-thread-3	 beforeInt：0	afterInt: 1
  pool-1-thread-1	 beforeInt：0	afterInt: 1
  pool-1-thread-3	 beforeInt：0	afterInt: 1
  ```

  

### 6.总结

- 因为每个Thread内有自己的**实例副本**且该副本只有当前线程自己使用
- 既然其他ThreadLocal不可访问，那就不存在多线程间共享问题
- 统一设置初始值，但是每个线程对这个值得修改都是各自线程互相独立得
- 如何才能不争抢

- - 加入synchronized或者Lock控制资源的访问顺序
  - 人手一份，大家各自安好，没有必要争抢

## 2.ThreadLocal源码分析

### 1.源码解读

### 2.Thread、ThreadLocal、ThreadLocalMap关系

- Thread和ThreadLocal，人手一份

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681367558745-dbaf5520-ddba-4900-8d0a-03b328d79141.png)

- ThreadLocal和ThreadLocalMap

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681367687947-8ac0ca12-440a-4945-bcdb-fbdfb5ad4fda.png)

- 三者总概括

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681367721060-7f26ea88-3756-4aab-8a78-76e87eb3827d.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

- - ThreadLocalMap实际上就是一个以ThreadLocal实例为Key，任意对象为value的Entry对象
  - 当我们为ThreadLocal变量赋值，实际上就是以当前ThreadLocal实例为Key，值为value的Entry往这个ThreadLocalMap中存放

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681370109524-7f3802f0-49be-4f8a-a9a3-bfd16c480eb1.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

### 3.总结

- ThreadLocalMap从字面上就可以看出这是一个保存ThreadLocal对象的map（其实是以ThreadLocal为Key），不过是经过了两层包装的ThreadLocal对象：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681368148461-8f9c0519-545d-447a-bc77-0ea31484ad59.png)

- **JVM内部维护了一个线程版的Map<ThreadLocal, Value>**（通过ThreadLocal对象的set方法，结果把ThreadLocal对象自己当作Key，放进了ThreadLocalMap中），每个线程要用到这个T的时候，用当前的线程去Map里面获取，**通过这样让每个线程都拥有了自己独立的变量**，人手一份，竞争条件被彻底消除，在并发模式下是绝对安全的变量。

## 3.ThreadLocal内存泄漏问题

### 1.什么是内存泄漏

不再会被使用的对象或者变量占用的内存不能被回收，就是内存泄漏

### 2.谁惹的祸？

- 再回首ThreadLocalMap

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681368935944-413631bd-34e1-4c46-9076-3b6e388f18f1.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

- 强软弱虚引用

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681368976782-7841e56a-c7fa-4bc6-9f93-e649ecef85df.png)

- - 强引用：

- - - 对于强引用的对象，**就算是出现了OOM也不会对该对象进行回收，死都不收**。在Java中最常见的就是强引用,把一个对象赋给一个引用变量,这个引用变量就是一个强引用。当一个对象被强引用变量引用时，它处于可达状态，是不可能被垃圾回收机制回收的，**即使该对象以后永远都不会被用到，JVM也不会回收**，因此强引用是造成Java内存泄露的主要原因之一。
    - 对于一个普通的对象,如果没有其他的引用关系,只要超过了引用的作用域或者显式地将相应(强)引用赋值为null,般认为就是可以被垃圾收集的了(当然具体回收时杋还是要看垃圾收集策略)。

- - 软引用：

- - - 是一种相对强引用弱化了一些的引用，对于只有软引用的对象而言，**当系统内存充足时，不会被回收，当系统内存不足时，他会被回收**，软引用通常用在对内存敏感的程序中，比如高速缓存，内存够用就保留，不够用就回收。

- - 弱引用：

- - - 比软引用的生命周期更短，对于只有弱引用的对象而言，只要垃圾回收机制一运行，**不管JVM的内存空间是否足够，都会回收该对象占用的内存**。

- - 软引用和弱引用的**使用场景**----->假如有一个应用需要读取大量的本地图片：

- - - 如果每次读取图片都从硬盘读取则会严重影响性能
    - 如果一次性全部加载到内存中又可能会造成内存溢出
    - 此时使用软引用来解决，**设计思路**：用一个HashMap来保存图片的路径和与相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占用的空间，有效避免了OOM的问题

- - 虚引用：

- - - **虚引用必须和引用队列联合使用**，**如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都有可能被垃圾回收器回收**，它不能单独使用也不能通过它访问对象。
    - 虚引用的主要作用是跟踪对象被垃圾回收的状态。**仅仅是提供了一种确保对象被finalize后，做某些事情的通知机制**。换句话说就是在对象被GC的时候会收到一个系统通知或者后续添加进一步的处理，用来实现比finalize机制更灵活的回收操作。
    - PhantomReference的get方法总是返回null

- GCRoots

  ![image-20231211162054934](../../../../../picbed/store/picbed/img/image-20231211162054934.png)

### 3.为什么要用弱引用？不用如何？

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681370661933-928e40a9-f163-482b-8c0e-998f1fc0d276.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

- 为什么要用弱引用：

- - 当方法执行完毕后，栈帧销毁，强引用t1也就没有了，但此时线程的ThreadLocalMap里某个entry的Key引用还指向这个对象，若这个Key是强引用，就会导致Key指向的ThreadLocal对象即V指向的对象不能被gc回收，造成内存泄露
  - 若这个引用时弱引用就大概率会减少内存泄漏的问题（当然，**还得考虑key为null这个坑**），使用弱引用就可以使ThreadLocal对象在方法执行完毕后顺利被回收且entry的key引用指向为null

- 这里有个需要注意的问题：

- - ThreadLocalMap使用ThreadLocal的弱引用作为Key，如果一个ThreadLocal没有外部强引用引用他，那么系统gc时，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现**Key为null的Entry**，就没有办法访问这些Key为null的Entry的value，**如果当前线程迟迟不结束的话（好比正在使用线程池），这些key为null的Entry的value就会一直存在一条强引用链**：Tread Ref——Thread——ThreadLocalMap——Entry——value，**永远无法回收，造成内存泄漏**。
  - 虽然弱引用，保证了Key指向的ThreadLocal对象能够被及时回收，但是v指向的value对象是需要ThreadLocalMap调用get、set时发现key为null时才会去回收整个entry、value，**因此弱引用不能100%保证内存不泄露，我们要在不使用某个ThreadLocal对象后，手动调用remove方法来删除它**，尤其是在线程池中，不仅仅是内存泄漏的问题，因为线程池中的线程是重复使用的，意味着这个线程的ThreadLocalMap对象也是重复使用的，如果我们不手动调用remove方法，那么后面的线程就有可能获取到上个线程遗留下来的value值，造成bug。
  - 清除脏Entry----key为null的entry

- - - set()方法

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681439164133-0c01db19-01cd-43ff-92fc-fb827c8927d8.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

- - - get()方法

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681439454200-887f0822-96d6-4060-ab08-e49f72383790.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

- - - remove()

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681439555545-01a892b9-b0f4-479c-8c4c-7e2dadc07d17.png)

都会通过expungeStaleEntry，replaceStaleEntry，cleanSomeSlots清理掉key为null的脏entry

### 4.最佳实践

- ThreadLocal一定要**初始化，避免空指针异常**。
- **建议把ThreadLocal修饰为static**——实现线程隔离，不在于它本身，在于ThreadLocalMap，可以只初始化一次，分配一块内存即可
- **用完记得手动remove**

## 4.小总结

- ThreadLocal并不解决线程间共享数据的问题
- ThreadLocal适用于变量在线程间隔离且在方法间共享的场景
- ThreadLocal通过隐式的在不同线程内创建独立实例副本避免了实例线程安全的问题
- 每个线程持有一个只属于它自己的专属map并维护了ThreadLocal对象与具体实例的映射，该Map由于只被持有他的线程访问，故不存在线程安全以及锁的问题
- ThreadLocalMap的Entry对ThreadLocal的引用为弱引用。避免了ThreadLocal对象无法被回收的问题
- 都会通过expungeStaleEntry，cleanSomeSlots，replaceStaleEntry这三个方法回收键为null的Entry对象的值（即为具体实例）以及entry对象本身从而防止内存泄漏，属于安全加固的方法