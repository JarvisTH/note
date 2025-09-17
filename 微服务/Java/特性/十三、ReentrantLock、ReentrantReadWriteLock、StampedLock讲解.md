## 1.关于锁的面试题

- 你知道Java里面有那些锁
- 你说说你用过的锁，锁饥饿问题是什么？
- 有没有比读写锁更快的锁
- StampedLock知道吗？（邮戳锁/票据锁）
- ReentrantReadWriteLock有锁降级机制，你知道吗？

## 2.简单聊聊ReentrantReadWriteLock

### 1.是什么？

- 读写锁说明

- - 一个资源能够被**多个读线程**访问，或者被**一个写线程**访问，但是不能同时存在读写线程

- 再说说演变

- - 无锁无序->加锁->读写锁->邮戳锁

- 读写锁意义和特点

- - 它只允许**读读共存**，而**读写和写写依然是互斥的**，大多实际场景是”读/读“线程间不存在互斥关系，只有”读/写“线程或者”写/写“线程间的操作是需要互斥的，因此**引入了 ReentrantReadWriteLock**
  - 一个ReentrantReadWriteLock同时只能存在一个写锁但是可以存在多个读锁，但是不能同时存在写锁和读锁，也即资源可以被多个读操作访问，或一个写操作访问，但两者不能同时进行。
  - 只有在读多写少情景之下，读写锁才具有较高的性能体现。

### 2.特点

- 可重入
- 读写兼顾
- 结论：一体两面，读写互斥，读读共享，读没有完成的时候其他线程写锁无法获得
- 锁降级：可以完成写后读

- - 将**写锁降级为读锁--**---->**遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级为读锁**
  - 如果**同一个线程持有了写锁**，在没有释放写锁的情况下，它还可以继续获得读锁。这就是写锁的降级，降级成为了读锁。
  - 如果释放了写锁，那么就完全转换为读锁
  - 如果有线程在读，那么写线程是无法获取写锁的，是悲观锁的策略

- ![image-20231214140510718](../../../../../picbed/store/picbed/img/image-20231214140510718.png)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35653686/1681719044891-56077a92-8f12-49e4-85df-bdffbc21b424.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0)

锁降级是为了让当前线程感知数据的变化，目的是保证**数据可见性**

```java
public class LockDownDemo {
    public static void main(String[] args) {
        ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

        ReentrantReadWriteLock.WriteLock writeLock = readWriteLock.writeLock();
        ReentrantReadWriteLock.ReadLock readLock = readWriteLock.readLock();

        writeLock.lock();
        System.out.println("--写入");

        readLock.lock();
        System.out.println("--读取");

        writeLock.unlock();
        readLock.unlock();
    }
}

--写入
--读取
```

Reentrantreadwritelock还是甚于AQS实现的,还是对 state进行操作,拿到锁资源就去干活,如果没有拿到,依然去AQS队列中排队。

读锁操作:基于 state的高16位进行操作。

写锁操作:基于 state的低16为进行操作。

Reentrantread Writelock依然是可重入锁。

写锁重入:读写锁中的写锁的重入方式,基本和 Reentrantlock致,没有什么区别,依然是对 state进行+1操作即可,只要确认持有锁资源的线程,是当前写锁线程即可。只不过之前 Reentrantlock的重入次数是 state的正数取值范围,但是读写锁中写锁范围就变小了。

读锁重入:因为读锁是共享锁。读锁在获取锁资源操作时,是要对 state的高16位进行+1操作。因为读锁是共享锁,所以同一时间会有多个读线程持有读锁资源。这样一来,多个读操作在持有读锁时,无法确认每个线程读重入的次数。为了去记录读锁重入的次数,每个读操作的线程,都会有个 Thread Loca记录锁重入的次数。



## 3.面试题：有没有比读写锁更快的锁？

## 4.邮戳锁StampedLock

### 1.是什么？

StampedLock是JDK1.8中新增的一个读写锁，也是对JDK1.5中的**读写锁ReentrantReadWriteLock的优化**

stamp 代表了锁的状态。当stamp返回零时，表示线程获取锁失败，并且当释放锁或者转换锁的时候，都要传入最初获取的stamp值。

### 2.它是由饥饿问题引出

- 锁饥饿问题：

- - ReentrantReadWriteLock实现了读写分离，但是**一旦读操作比较多的时候，想要获取写锁就变得比较困难了，因此当前有可能会一直存在读锁，而无法获得写锁。**

- 如何解决锁饥饿问题：

- - 使用”**公平“策略**可以一定程度上缓解这个问题
  - 使用”公平“策略是**以牺牲系统吞吐量**为代价的
  - **StampedLock**类的**乐观读锁**方式--->采取乐观获取锁，**其他线程尝试获取写锁时不会被阻塞，在获取乐观读锁后，还需要对结果进行校验**

### 3.StampedLock的特点

- 所有**获取锁**的方法，都返回一个邮戳，stamp为零表示失败，其余都表示成功
- 所有**释放锁**的方法，都需要一个邮戳，这个stamp必须是和成功获取锁时得到的stamp一致
- StampedLock是**不可重入**的，危险（如果一个线程已经持有了写锁，在去获取写锁的话**会造成死锁**）
- StampedLock有三种访问模式：

- - **Reading（读模式悲观）**：功能和ReentrantReadWriteLock的读锁类似
  - **Writing（写模式）**：功能和ReentrantReadWriteLock的写锁类似
  - **Optimistic reading（乐观读模式）**：无锁机制，类似与数据库中的乐观锁，支持读写并发，**很乐观认为读时没人修改，假如被修改再实现升级为悲观读模式**

- 一句话：**读的过程中也允许写锁介入**

### 4.乐观读模式Code演示

- 传统的读写锁模式----读的时候写锁不能获取

```java
public class StampedLockDemo {
    static int number = 37;
    static StampedLock stampedLock = new StampedLock();

    public void write() {
        long stamp = stampedLock.writeLock();
        System.out.println(Thread.currentThread().getName() + "\t" + "写线程准备修改");
        try {
            number = number + 13;
        } finally {
            stampedLock.unlockWrite(stamp);
        }
        System.out.println(Thread.currentThread().getName() + "\t" + "写线程结束修改");
    }

    public void read() {
        long stamp = stampedLock.readLock();
        System.out.println(Thread.currentThread().getName() + "\t" + " come in readLock codeBlock");
        for (int i = 0; i < 4; i++) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + " 正在读取中");
        }
        try {
            int result = number;
            System.out.println(Thread.currentThread().getName() + "\t" + "获得成员变量值result: " + result);
            System.out.println("写线程没有修改成功，读锁时候写锁无法介入，传统的读写互斥");
        } finally {
            stampedLock.unlockRead(stamp);
        }

    }

    public static void main(String[] args) {
        StampedLockDemo resource = new StampedLockDemo();
        new Thread(() -> {
            resource.read();
        }, "readThread").start();

        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"\t"+" come in");
            resource.write();
        }, "writeThread").start();
    }
}
/**
 * readThread	 come in readLock codeBlock
 * readThread	 正在读取中
 * writeThread	 come in
 * readThread	 正在读取中
 * readThread	 正在读取中
 * readThread	 正在读取中
 * readThread	获得成员变量值result: 37
 * 写线程没有修改成功，读锁时候写锁无法介入，传统的读写互斥
 * writeThread	写线程准备修改
 * writeThread	写线程结束修改
 */
```

- 乐观读模式----读的过程中也允许写锁介入

```java
public class StampedLockDemo {
    static int number = 37;
    static StampedLock stampedLock = new StampedLock();

    public void write() {
        long stamp = stampedLock.writeLock();
        System.out.println(Thread.currentThread().getName() + "\t" + "写线程准备修改");
        try {
            number = number + 13;
        } finally {
            stampedLock.unlockWrite(stamp);
        }
        System.out.println(Thread.currentThread().getName() + "\t" + "写线程结束修改");
    }

    public void read() {
        long stamp = stampedLock.tryOptimisticRead();

        int result = number;

        System.out.println("4秒前 stampedLock.validate方法值（true 无修改 false有修改）" + "\t" + stampedLock.validate(stamp));
        for (int i = 0; i < 4; i++) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + " 正在读取...." + i + "秒后" + "stampedLock.validate方法值（true 无修改 false有修改）" + "\t" + stampedLock.validate(stamp));
        }
        if (!stampedLock.validate(stamp)) {
            System.out.println("有人修改----------有写操作");
            stamp = stampedLock.readLock();
            try {
                System.out.println("从乐观读升级为悲观读");
                result = number;
                System.out.println("重新悲观读后result：" + result);
            } finally {
                stampedLock.unlockRead(stamp);
            }
        }
        System.out.println(Thread.currentThread().getName() + "\t" + "finally value: " + result);

    }


    public static void main(String[] args) {
        StampedLockDemo resource = new StampedLockDemo();
        new Thread(() -> {
            resource.read();
        }, "readThread").start();

        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t" + " come in");
            resource.write();
        }, "writeThread").start();
    }
}
/**
 * 4秒前 stampedLock.validate方法值（true 无修改 false有修改）	true
 * readThread	 正在读取....0秒后stampedLock.validate方法值（true 无修改 false有修改）	true
 * readThread	 正在读取....1秒后stampedLock.validate方法值（true 无修改 false有修改）	true
 * writeThread	 come in
 * writeThread	写线程准备修改
 * writeThread	写线程结束修改
 * readThread	 正在读取....2秒后stampedLock.validate方法值（true 无修改 false有修改）	false
 * readThread	 正在读取....3秒后stampedLock.validate方法值（true 无修改 false有修改）	false
 * 有人修改----------有写操作
 * 从乐观读升级为悲观读
 * 重新悲观读后result：50
 * readThread	finally value: 50
 */
```

### 6.StampedLock的缺点

- StampedLock不支持重入，没有Re开头
- StampedLock的悲观读锁和写锁都**不支持条件变量（condition）**，这个也需要注意
- 使用StampedLock**一定不要调用中断操作**，即不要调用interrupt()方法

