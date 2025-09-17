## 1.面试题

- 谈谈你对Synchronized的理解
- Sychronized的锁升级你聊聊
- Synchronized实现原理，monitor对象什么时候生成的？知道monitor的monitorenter和monitorexit这两个是怎么保证同步的嘛？或者说这两个操作计算机底层是如何执行的
- 偏向锁和轻量级锁有什么区别

## 2.Synchronized的性能变化

- Java5以前，只有Synchronized，这个是操作系统级别的重量级操作

- - 重量级锁，假如锁的竞争比较激烈的话，性能下降
  - Java 5之前 用户态和内核态之间的转换

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681448109572-488801b7-7a7c-4003-88f8-a10c59b3859d.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

- Java6 之后为了减少获得锁和释放锁所带来的性能消耗，引入了轻量级锁和偏向锁

- java对象是天生的monitor，因为设计中每个Java对象自带一个看不见的锁，即内部锁或者monitor锁。monitor本质依赖于底层操作系统的Mutex Lock实现。JVM中的同步基于进入和退出管程（monitor）对象实现的，每个对象实例都会有一个monitor，monitor由ObjectMonitor实现。

  ![image-20231212171753014](../../../../../picbed/store/picbed/img/image-20231212171753014.png)

- Monitor与java对象以及线程如何关联？

  - 如果一个java对象被某个线程锁住，该java对象的MarkWord字段中LockWord指向monitor的起始地址
  - monitor的Owner字段存放拥有相关联对象锁的线程id

![image-20231212172039183](../../../../../picbed/store/picbed/img/image-20231212172039183.png)

![image-20231212172236599](../../../../../picbed/store/picbed/img/image-20231212172236599.png)



## 3.Synchronized锁种类及升级步骤

### 1.多线程访问情况

- 只有一个线程来访问，有且唯一Only One
- 有两个线程（2个线程交替访问）
- 竞争激烈，更多线程来访问

### 2.升级流程

- Synchronized用的锁是存在Java对象头里的MarkWord中，**锁升级功能**主要依赖MarkWord中**锁标志位和释放偏向锁标志位**

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681448615179-68918d36-eeaa-4454-b672-0d2eb1ae30fe.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

- 锁指向，请牢记

- - 偏向锁：MarkWord存储的是**偏向的线程ID**
  - 轻量锁：MarkWord存储的是指向**线程栈**中Lock Record的指针
  - 重量锁：MarkWord存储的是指向**堆中**的monitor对象（系统互斥量指针）

![image-20231212170647982](../../../../../picbed/store/picbed/img/image-20231212170647982.png)

### 3.无锁

```java
public class SynchronizedUpDemo {
    public static void main(String[] args) {
        Object o = new Object();
        System.out.println(Integer.toHexString(o.hashCode()));
        System.out.println(ClassLayout.parseInstance(o).toPrintable());
    }
}

1554909b
java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000001554909b01 (hash: 0x1554909b; age: 0)
  8   4        (object header: class)    0x00001000
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

无锁状态：01

### 4.偏锁(JDK15已废除)

偏向锁：**单线程竞争**，当线程A第一次竞争到锁时，通过修改MarkWord中的偏向线程ID、偏向模式。如果不存在其他线程竞争，那么持有偏向锁的线程将永远不需要进行同步。

主要作用：

- **当一段同步代码一直被同一个线程多次访问，由于只有一个线程那么该线程在后续访问时便会自动获得锁**
- 同一个老顾客来访，直接老规矩行方便

结论：

- HotSpot的作者经过研究发现，大多数情况下：在多线程情况下，锁不仅不存在多线程竞争，还存在由同一个线程多次获得的情况，偏向锁就是在这种情况下出现的，它的出现是为了**解决只有一个线程执行同步时提高性能。**
- 偏向锁会偏向于第一个访问锁的线程，如果在接下来的运行过程中，该锁没有被其他线程访问，则持有偏向锁的线程将永远不需要出发同步。也即偏向锁在资源在没有竞争情况下消除了同步语句，懒得连CAS操作都不做了，直接提高程序性能。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681451836192-5ae30f1e-085d-4c2b-aa8c-1bd47f199116.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681451914425-01aa0721-8f56-4413-a1db-5092fcbfdf06.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

偏向锁JVM命令：

- java -XX:+PrintFlagsInitial|grep BiasedLock*

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681452242110-e3b8e768-acfb-4f43-9a50-8699e9952ece.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

![image-20231213122651765](../../../../../picbed/store/picbed/img/image-20231213122651765.png)

```java
public class SynchronizedUpDemo {
    public static void main(String[] args) {
        Object o = new Object();
        synchronized (o) {
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }
    }
}

java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x000002f16563f805 (biased: 0x00000000bc5958fe; epoch: 0; age: 0)
  8   4        (object header: class)    0x00001000
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
    
    
public class SynchronizedUpDemo {
    public static void main(String[] args) {
        Object o = new Object();
        System.out.println(ClassLayout.parseInstance(o).toPrintable());
    }
}
java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000000000000005 (biasable; age: 0)
  8   4        (object header: class)    0x00001000
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

0x000002f16563f805  ： 05转为2进制为00000101，偏向锁，其他值为线程id指针

0x0000000000000005：偏向锁，由于o对象没有使用synchronized加锁，所以线程id是空的。

偏向锁的撤销：

- 当有另外一个线程逐步来竞争锁的时候，就不能再使用偏向锁了，要升级为轻量级锁，**使用的是等到竞争出现才释放锁的机制**
- 竞争线程尝试CAS更新对象头失败，会等到**全局安全点（此时不会执行任何代码）**撤销偏向锁，同时检查持有偏向锁的线程是否还在执行：

- - 第一个线程正在执行Synchronized方法（处于同步块），它还没有执行完，其他线程来抢夺，该偏向锁会被取消掉并出现锁升级，此时轻量级锁由原来持有偏向锁的线程持有，继续执行同步代码块，而正在竞争的线程会自动进入自旋等待获得该轻量级锁
  - 第一个线程执行完Synchronized（退出同步块），则将对象头设置为无所状态并撤销偏向锁，重新偏向。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681453654704-4252f703-ba8d-43c1-90b6-cdf78a965570.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

题外话：Java15以后逐步废弃偏向锁，需要手动开启------->维护成本高

### 5.轻锁

概念：多线程竞争，但是任意时候最多只有一个线程竞争，即不存在锁竞争太激烈的情况，也就没有线程阻塞。

主要作用：有线程来参与锁的竞争，但是获取锁的冲突时间极短---------->本质是**自旋锁CAS**。

轻量级锁是为了在线程**近乎交替**执行同步块时提高性能。

主要目的:在没有多线程竞争的前提下,通过CAS减少重量级锁使用操作系统互斥量产生的性能消耗,说白了先自旋,不行オ升级阻塞。

升级时机:当关闭偏向锁功能或多线程竟争偏向锁会导致偏向锁升级为轻量级锁

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681454510407-cde9df3c-e7ed-415f-a459-76f64160035f.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

轻量锁的获取：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681454665890-db296f20-69b8-4bd8-8538-0197584c61e5.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681454779796-56f6d99a-8850-44da-ad34-890a738a230c.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

Demo：-XX:-UseBiasedLocking，关闭偏向锁直接进入轻量级锁

```java
public class SynchronizedUpDemo {
    public static void main(String[] args) {
        Object o = new Object();
        new Thread(()->{
            synchronized (o){
                System.out.println(ClassLayout.parseInstance(o).toPrintable());
            }
        },"t1").start();
    }
}

java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x000000e23bcfeff8 (thin lock: 0x000000e23bcfeff8)
  8   4        (object header: class)    0x00001000
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

可以看到进入到了轻量级锁 thin lock，8转为2进制为1000。

自旋一定程度和次数（Java8 之后是自适应自旋锁------意味着自旋的次数不是固定不变的）：

- 线程如果自旋成功了，那下次自旋的最大次数会增加，因为JVM认为既然上次成功了，那么这一次也大概率会成功
- 如果很少会自选成功，那么下次会减少自旋的次数甚至不自旋，避免CPU空转

轻量锁和偏向锁的区别：

- 争夺轻量锁失败时，自旋尝试抢占锁
- 轻量级锁每次退出同步块都需要释放锁，而偏向锁是在竞争发生时才释放锁

### 6.重锁

有大量线程参与锁的竞争，冲突性很高

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681455335764-3bb5bb5f-d014-4b25-9755-e0ca2c9aa809.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

### 7.总结

#### 1.锁升级过程

![20200602120540100.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/35653686/1681459024640-cadca197-5d19-433d-b6d8-abc8965ce494.jpeg)

#### 2.锁升级后，hashcode去哪儿了?

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681448615179-68918d36-eeaa-4454-b672-0d2eb1ae30fe.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681455960993-7d1f944c-3f3e-47ad-a7e3-83abedc0a8fe.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681455773115-6a73a005-970b-4450-a264-cdec59354184.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

```java
public class SynchronizedUpDemo {
    public static void main(String[] args) {
        Object o = new Object();
        synchronized (o){
            System.out.println(o.hashCode());
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }
    }
}

java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000028afe9ca502 (fat lock: 0x0000028afe9ca502)
  8   4        (object header: class)    0x00001000
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

而获取了hascode后，直接进入了重锁。

```java
public class SynchronizedUpDemo {
    public static void main(String[] args) {
        Object o = new Object();
        System.out.println(ClassLayout.parseInstance(o).toPrintable());
        System.out.println(o.hashCode());
        synchronized (o) {
            System.out.println(ClassLayout.parseInstance(o).toPrintable());
        }
    }
}

java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x0000000000000005 (biasable; age: 0)
  8   4        (object header: class)    0x00001000
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

415186196
java.lang.Object object internals:
OFF  SZ   TYPE DESCRIPTION               VALUE
  0   8        (object header: mark)     0x000000dabfbff6c0 (thin lock: 0x000000dabfbff6c0)
  8   4        (object header: class)    0x00001000
 12   4        (object alignment gap)    
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

偏向锁状态时获取了hascode，后面升级为轻量级锁。

#### 3.各种锁优缺点、synchronized锁升级和实现原理

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681456189232-f2cbc220-e6bc-4854-9a5d-e963623a860a.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

## 4.JIT编译器对锁的优化

### 1.JIT

Just In Time Compiler 即时编译器

### 2.锁消除

```java
public class LockBigDemo {
    static Object object = new Object();

    public void m1() {
        Object o = new Object();
        synchronized (o) {
            System.out.println("--LockBigDemo" + "\t" + o.hashCode() + "\t" + object.hashCode());
        }
    }

    public static void main(String[] args) {
        LockBigDemo lockBigDemo = new LockBigDemo();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                lockBigDemo.m1();
            }, String.valueOf(i)).start();
        }
    }
}

--LockBigDemo	1801470815	1705484355
--LockBigDemo	387335703	1705484355
--LockBigDemo	1906000844	1705484355
--LockBigDemo	1539378682	1705484355
--LockBigDemo	1350047910	1705484355
--LockBigDemo	168734963	1705484355
--LockBigDemo	473460248	1705484355
--LockBigDemo	364904459	1705484355
--LockBigDemo	391918859	1705484355
--LockBigDemo	637558281	1705484355
```

出现锁消除问题，JIT编译器会忽略synchronized (o)，每次new处理的不存在了，非正常的，底层没有加锁。

### 3.锁粗化

```java
public class LockBigDemo {
    static Object objectLock = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (objectLock) {
                System.out.println("111111111111");
            }
            synchronized (objectLock) {
                System.out.println("222222222222");
            }
            synchronized (objectLock) {
                System.out.println("333333333333");
            }
            synchronized (objectLock) {
                System.out.println("444444444444");
            }
            //底层JIT的锁粗化优化
            synchronized (objectLock) {
                System.out.println("111111111111");
                System.out.println("222222222222");
                System.out.println("333333333333");
                System.out.println("444444444444");
            }
        }, "t1").start();
    }
}
```

假如方法中首尾相接，前后相邻的都是同一个锁对象，那JIT编译器会把这几个synchronized块合并为一个大块加粗加大范围，一次申请锁使用即可，避免次次的申请和释放锁，提高了性能。