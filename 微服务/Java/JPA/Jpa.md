## 一、简介

- 定义在javax,persistence包下面的API标准
- 执行crud操作，利用hibernate
- 定义了一套基于对象的SQL：JPQL
- 写面向对象（JPQL）而非面向数据库查询语言
- 直接通过注解方式表示java实体对象及元数据对象和数据表间的映射关系
- 通过操作session中不同实体状态实现DB同步

![image-20220302235815088](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220302235824.png)

## 二、Repository

顶级父类接口，DB操作入口类

![image-20220303000325603](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220303000327.png)

- ReactiveCrudRepository：响应式编程，对应nosql方面的操作
- CrudRepository：jpa操作
- RxJava2CrudRepository：支持RxJava2
- CoroutineCrudRepository：支持kotin语法

<img src="https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220303001226.png" alt="image-20220303001211347" style="zoom: 200%;" />

Repository接口

- Repository

- CrudRepository：简单crud方法

  <img src="https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220303001746.png" alt="image-20220303001740799" style="zoom:150%;" />

- PagingAndSortingRepository：带分页排序方法

  <img src="https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220303002033.png" alt="image-20220303002028966" style="zoom:150%;" />

- QueryByExampleExcutor，带简单Example方法

  <img src="../../../../picbed/store/picbed/img/image-20220303002058554.png" alt="image-20220303002058554" style="zoom:150%;" />

- JpaRepository，JPA扩展方法

  ![image-20220303002257070](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220303002258.png)

- JpaSpecificationExecutor，JpaSpecificatio扩展查询

  <img src="../../../../picbed/store/picbed/img/image-20220303002511105.png" alt="image-20220303002511105" style="zoom:150%;" />

- QueryDslPredicateExecutor，QueryDsl封装

实现类：

- SimpleJpaRepository：JPA所有接口的默认实现类
- QueryDslJpaRepository：QueryDsl封装

使用是根据实际场景继承不同接口，理解动态代理的作用。

## 三、查询方法的命名方法和参数

利用方法名定义查询方法做crud。

- 直接通过方法名实现
- @Query手动在方法上定义

### 1.配置与使用

只需要继承spring data common里面的任意Repository接口或者自定义接口，可以选择性的暴露父接口方法。

<img src="https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220303003128.png" style="zoom:80%;" />

### 2.方法查询策略

@EnableJpaRepositories注解配置查询策略