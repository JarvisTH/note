https://www.freesion.com/article/6425514764/

## 一、**Criteria 查询基本概念**

Spring Data JPA 支持 JPA2.0 的 Criteria 查询，相应的接口是 JpaSpecificationExecutor。Criteria 查询：是一种类型安全和更面向对象的查询 。这个接口基本是围绕着 Specification 接口来定义的， Specification 接口中只定义了如下一个方法：

```java
public interface JpaSpecificationExecutor<T> {
    Optional<T> findOne(@Nullable Specification<T> spec);

    List<T> findAll(@Nullable Specification<T> spec);

    Page<T> findAll(@Nullable Specification<T> spec, Pageable pageable);

    List<T> findAll(@Nullable Specification<T> spec, Sort sort);

    long count(@Nullable Specification<T> spec);
}
```

```java
Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb); 
```

Criteria 查询是以元模型的概念为基础的，元模型是为具体持久化单元的受管实体定义的，这些实体可以是实体类，嵌入类或者映射的父类。提供受管实体元信息的类就是元模型类。

使用元模型类最大的优势是凭借其实例化可以在编译时访问实体的持久属性。该特性使得 criteria 查询更加类型安全.

```java
@Entity
@Table
public class Employee{  
    private int id;   
    private String name;
    private int age;
    @OneToMany
    private List<Address> addresses;
    // Other code…
}
```

```java
import javax.annotation.Generated;
import javax.persistence.metamodel.SingularAttribute;
import javax.persistence.metamodel.ListAttribute;
import javax.persistence.metamodel.StaticMetamodel;
@StaticMetamodel(Employee.class)
public class Employee_ {     
    public static volatile SingularAttribute<Employee, Integer> id;   
    public static volatile SingularAttribute<Employee, Integer> age;   
    public static volatile SingularAttribute<Employee, String> name;    
    public static volatile ListAttribute<Employee, Address> addresses;
}
```

Employee的每一个属性都会使用在JPA2规范中描述的以下规则在相应的元模型类中映射：

```java
元模型类的属性全部是static和public的。Employee的每一个属性都会使用在JPA2规范中描述的以下规则在相应的元模型类中映射：
对于Addess这样的集合类型，会定义静态属性ListAttribute< A, B> b，这里List对象b是定义在类A中类型B的对象。其它集合类型可以是SetAttribute, MapAttribute 或 CollectionAttribute 类型。
```

**为什么要使用元模型，答：查询类型更加安全。**

## CRITERIABUILDER 安全查询创建工厂

CriteriaBuilder 安全查询创建工厂，创建 **CriteriaQuery**，创建查询具体条件 **Predicate** 等。

CriteriaBuilder是一个工厂对象,安全查询的开始.用于构建JPA安全查询.可以从EntityManager 或 EntityManagerFactory类中获得CriteriaBuilder。

```java
CriteriaBuilder criteriaBuilder = em.getCriteriaBuilder();
```

## CRITERIAQUERY 安全查询主语句

- 它通过调用 CriteriaBuilder, createQuery 或CriteriaBuilder.createTupleQuery 获得。
- CriteriaBuilder就像CriteriaQuery 的工厂一样。
- CriteriaQuery对象必须在实体类型或嵌入式类型上的Criteria 查询上起作用。
- Employee实体的 CriteriaQuery 对象以下面的方式创建：

```java
CriteriaBuilder criteriaBuilder = em.getCriteriaBuilder();
CriteriaQuery<Employee> criteriaQuery = criteriaBuilder.createQuery(Employee.class);
```

## ROOT 定义查询的 FROM 子句中能出现的类型

![image-20230301211736112](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230301211736112.png)

查询表达式被赋予泛型。一些典型的表达式是：

- Root<T>, 相当于一个 From 子句,定义查询的 From 子句中能出现的类型
- Criteria查询的查询根定义了实体类型，能为将来导航获得想要的结果，它与SQL查询中的FROM子句类似。
- Root实例也是类型化的，且定义了查询的FROM子句中能够出现的类型。
- 查询根实例能通过传入一个实体类型给 AbstractQuery.from方法获得
- Criteria查询，可以有多个查询根。
- Employee 实体的查询根对象可以用以下语法获得

```java
Root<Employee> employee = criteriaQuery.from(Employee.class);
```

```java
package javax.persistence.criteria;

import javax.persistence.metamodel.EntityType;

public interface Root<X> extends From<X, X> {
    EntityType<X> getModel();
}
```

可以看出 Root 继承自 **From**

```java
public interface From<Z, X> extends Path<X>, FetchParent<Z, X>
```

而 From 又继承自 **Path**

```java
public interface Path<X> extends Expression<X> {
    Bindable<X> getModel();

    Path<?> getParentPath();

    <Y> Path<Y> get(SingularAttribute<? super X, Y> var1);

    <E, C extends Collection<E>> Expression<C> get(PluralAttribute<X, C, E> var1);

    <K, V, M extends Map<K, V>> Expression<M> get(MapAttribute<X, K, V> var1);

    Expression<Class<? extends X>> type();

    <Y> Path<Y> get(String var1);
}
```

## PREDICATE 过滤条件

- 过滤条件应用到SQL语句的FROM子句中。
- 在criteria 查询中，查询条件通过Predicate 或Expression 实例应用到CriteriaQuery 对象上。
- 这些条件使用 CriteriaQuery .where 方法应用到CriteriaQuery 对象上。
- Predicate 实例也可以用Expression 实例的 isNull， isNotNull 和 in方法获得，复合的Predicate 语句可以使用CriteriaBuilder的and, or andnot 方法构建。
- CriteriaBuilder 也是作为Predicate 实例的工厂，Predicate 对象通过调用CriteriaBuilder 的条件方法（ equal，notEqual， gt， ge，lt， le，between，like等）创建。
- 这些条件使用 CriteriaQuery .where 方法应用到CriteriaQuery 对象上。

```java
Predicate condition = criteriaBuilder.gt(employee.get(Employee_.age), 24);
criteriaQuery.where(condition);
```

### Predicate[] 多个过滤条件

```java
List<Predicate> predicatesList = new ArrayList<Predicate>();
predicatesList.add(.....Pridicate....)
criteriaQuery.where(predicatesList.toArray(new Predicate[predicatesList.size()]));
```

OR 语句

```java
predicatesList.add(
    criteriaBuilder.or(
        criteriaBuilder.equal(root.get(RepairOrder_.localRepairStatus),LocalRepairStatus.repairing),
        criteriaBuilder.equal(root.get(RepairOrder_.localRepairStatus), LocalRepairStatus.diagnos)
    )
 );
```

忽略大小写 (全大写)

```java
predicatesList.add(
    criteriaBuilder.like(
       criteriaBuilder.upper(root.get(RepairShop_.shopName)),
       StringUtils.upperCase(StringUtils.trim(this.shopName)) + "%")
);
```

## TypedQuery执行查询与获取元模型实例

使用 EntityManager 创建查询时，可以在输入中指定一个 CriteriaQuery 对象，它返回一个 TypedQuery，它是 JPA 2.0 引入 javax.persistence.Query 接口的一个扩展，TypedQuery 接口知道它返回的类型。

所以使用中，先创建查询得到 TypedQuery, 然后通过 typeQuery 得到结果。

当 EntityManager.createQuery (CriteriaQuery) 方法调用时，一个可执行的查询实例会创建，该方法返回指定从 criteria 查询返回的实际类型的 TypedQuery 对象。TypedQuery 接口是 javax.persistence.Queryinterface. 的子类型。在该片段中， TypedQuery 中指定的类型信息是 Employee，调用 getResultList 时，查询就会得到执行 。

```java
TypedQuery<Employee> typedQuery = em.createQuery(criteriaQuery);
List<Employee> result = typedQuery.getResultList();
```

元模型实例通过调用 EntityManager.getMetamodel 方法获得，EntityType<Employee> 的元模型实例通过调用 Metamodel.entity (Employee.class) 而获得，其被传入 CriteriaQuery.from 获得查询根。

```java
Metamodel metamodel = em.getMetamodel();EntityType<Employee> 
Employee_ = metamodel.entity(Employee.class);
Root<Employee> empRoot = criteriaQuery.from(Employee_);
```

也有可能调用 Root.getModel 方法获得元模型信息。类型 EntityType<Dept> 的实例 Dept_和 name 属性可以调用 getSingularAttribute 方法获得，它与 String 文本进行比较：

```java
CriteriaQuery criteriaQuery = criteriaBuilder.createQuery();
Root<Dept> dept = criteriaQuery.from(Dept.class);
EntityType<Dept> Dept_ = dept.getModel();
Predicate testCondition = criteriaBuilder.equal(dept.get(Dept_.getSingularAttribute("name", String.class)), "Ecomm");
```

## Expression 用在查询语句的 select，where 和 having 子句中，该接口有 isNull, isNotNull 和 in 方法

Expression 对象用在查询语句的 select，where 和 having 子句中，该接口有 isNull, isNotNull 和 in 方法，下面的代码片段展示了 Expression.in 的用法，employye 的年龄检查在 20 或 24 的。

```java
CriteriaQuery<Employee> criteriaQuery = criteriaBuilder .createQuery(Employee.class);
Root<Employee> employee = criteriaQuery.from(Employee.class);
criteriaQuery.where(employee.get(Employee_.age).in(20, 24));
em.createQuery(criteriaQuery).getResultList();
```

或者

```java
//定义一个Expression
Expression<String> exp = root.get(Employee.id);

List<String> strList=new ArrayList<>();	
strList.add("20");
strList.add("24");		
predicatesList.add(exp.in(strList));
criteriaQuery.where(predicatesList.toArray(new Predicate[predicatesList.size()]));
```

## **复合谓词**

在 SQL 中，连接跨多张表以获取查询结果，类似的实体连接通过调用 From.join 执行，连接帮助从一个实体导航到另一个实体以获得查询结果。
Root 的 join 方法返回一个 Join<Dept, Employee> 类型 (也可以是 SetJoin,，ListJoin，MapJoin 或者 CollectionJoin 类型)。

默认情况下，连接操作使用内连接，而外连接可以通过在 join 方法中指定 JoinType 参数为 LEFT 或 RIGHT 来实现。

```java
CriteriaQuery<Dept> cqDept = criteriaBuilder.createQuery(Dept.class);
Root<Dept> deptRoot = cqDept.from(Dept.class);
Join<Dept, Employee> employeeJoin = deptRoot.join(Dept_.employeeCollection);
cqDept.where(criteriaBuilder.equal(employeeJoin.get(Employee_.deptId).get(Dept_.id), 1));
TypedQuery<Dept> resultDept = em.createQuery(cqDept);
```

## **抓取连接**

当涉及到 collection 属性时，抓取连接对优化数据访问是非常有帮助的。**这是通过预抓取关联对象和减少懒加载开销而达到的。**
使用 criteria 查询，fetch 方法用于指定关联属性
Fetch 连接的语义与 Join 是一样的，因为 Fetch 操作不返回 Path 对象，所以它不能将来在查询中引用。
在以下例子中，查询 Dept 对象时 employeeCollection 对象被加载，这不会有第二次查询数据库，因为有懒加载。

```java
CriteriaQuery<Dept> d = cb.createQuery(Dept.class);
Root<Dept> deptRoot = d.from(Dept.class);
deptRoot.fetch("employeeCollection", JoinType.LEFT);
d.select(deptRoot);
List<Dept> dList = em.createQuery(d).getResultList();
```

对应 SQL: SELECT * FROM dept d, employee e WHERE d.id = e.deptId

## **路径表达式**

Root 实例，Join 实例或者从另一个 Path 对象的 get 方法获得的对象使用 get 方法可以得到 Path 对象，当查询需要导航到实体的属性时，路径表达式是必要的。
Get 方法接收的参数是在实体元模型类中指定的属性。
Path 对象一般用于 Criteria 查询对象的 select 或 where 方法。例子如下：

```java
CriteriaQuery<String> criteriaQuery = criteriaBuilder.createQuery(String.class);
Root<Dept> root = criteriaQuery.from(Dept.class);
criteriaQuery.select(root.get(Dept_.name));
```

## **参数化表达式**

在 JPQL 中，查询参数是在运行时通过使用命名参数语法 (冒号加变量，如 :age) 传入的。在 Criteria 查询中，查询参数是在运行时创建 ParameterExpression 对象并为在查询前调用 TypeQuery,setParameter 方法设置而传入的。下面代码片段展示了类型为 Integer 的 ParameterExpression age，它被设置为 24：

```java
ParameterExpression<Integer> age = criteriaBuilder.parameter(Integer.class);
Predicate condition = criteriaBuilder.gt(testEmp.get(Employee_.age), age);
criteriaQuery.where(condition);
TypedQuery<Employee> testQuery = em.createQuery(criteriaQuery);
List<Employee> result = testQuery.setParameter(age, 24).getResultList();
Corresponding SQL: SELECT * FROM Employee WHERE age = 24;
```

## **排序结果**

Criteria 查询的结果能调用 CriteriaQuery.orderBy 方法排序，该方法接收一个 Order 对象做为参数。通过调用  CriteriaBuilder.asc 或 CriteriaBuilder.Desc，Order 对象能被创建。以下代码片段中，Employee 实例是基于 age 的升序排列。 

```java
CriteriaQuery<Employee> criteriaQuery = criteriaBuilder .createQuery(Employee.class);
Root<Employee> employee = criteriaQuery.from(Employee.class);
criteriaQuery.orderBy(criteriaBuilder.asc(employee.get(Employee_.age)));
em.createQuery(criteriaQuery).getResultList();
```

对应 SQL: SELECT * FROM Employee ORDER BY age ASC

## **分组**

CriteriaQuery 实例的 groupBy 方法用于基于 Expression 的结果分组。查询通过设置额外表达式，以后调用 having 方法。下面代码片段中，查询按照 Employee 类的 name 属性分组，且结果以字母 N 开头：

```java
CriteriaQuery<Tuple> cq = criteriaBuilder.createQuery(Tuple.class);
Root<Employee> employee = cq.from(Employee.class);
cq.groupBy(employee.get(Employee_.name));
cq.having(criteriaBuilder.like(employee.get(Employee_.name), "N%"));
cq.select(criteriaBuilder.tuple(employee.get(Employee_.name),criteriaBuilder.count(employee)));
TypedQuery<Tuple> q = em.createQuery(cq);
List<Tuple> result = q.getResultList();
```

对应 SQL:  SELECT name, COUNT(*) FROM employeeGROUP BY name HAVING name like 'N%'

## **查询投影**

Criteria 查询的结果与在 Critiria 查询创建中指定的一样。结果也能通过把查询根传入 CriteriaQuery.select 中显式指定。Criteria 查询也给开发者投影各种结果的能力。

## **使用 construct ()**

使用该方法，查询结果能由非实体类型组成。在下面的代码片段中，为 EmployeeDetail 类创建了一个 Criteria 查询对象，而 EmployeeDetail 类并不是实体类型。

```java
CriteriaQuery<EmployeeDetails> criteriaQuery = criteriaBuilder.createQuery(EmployeeDetails.class);
Root<Employee> employee = criteriaQuery.from(Employee.class);
criteriaQuery.select(criteriaBuilder.construct(EmployeeDetails.class,employee.get(Employee_.name),employee.get(Employee_.age)));
em.createQuery(criteriaQuery).getResultList();
```

## **返回 Object [] 的查询**

Criteria 查询也能通过设置值给 CriteriaBuilder.array 方法返回 Object [] 的结果。下面的代码片段中，数组大小是 2（由 String 和 Integer 组成）。

```java
CriteriaQuery<Object[]> criteriaQuery = criteriaBuilder.createQuery(Object[].class);
Root<Employee> employee = criteriaQuery.from(Employee.class);
criteriaQuery.select(criteriaBuilder.array(employee.get(Employee_.name), employee.get(Employee_.age)));
em.createQuery(criteriaQuery).getResultList();
```

对应 SQL: SELECT name, age FROM employee

## **返回元组 (Tuple) 的查询**

数据库中的一行数据或单个记录通常称为元组。通过调用 CriteriaBuilder.createTupleQuery () 方法，查询可以用于元组上。CriteriaQuery.multiselect 方法传入参数，它必须在查询中返回。

```java
CriteriaQuery<Tuple> criteriaQuery = criteriaBuilder.createTupleQuery();
Root<Employee> employee = criteriaQuery.from(Employee.class);
criteriaQuery.multiselect(employee.get(Employee_.name).alias("name"), employee.get(Employee_.age).alias("age"));
em.createQuery(criteriaQuery).getResultList();
```

对应 SQL: SELECT name, age FROM employee。



CriteriaQuery 接口：代表一个 specific 的顶层查询对象，它包含着查询的各个部分，比如：select 、from、where、group by、order by 等，CriteriaQuery 对象只对实体类型或嵌入式类型的 Criteria 查询起作用。

oot 接口：代表 Criteria 查询的根对象，Criteria 查询的查询根定义了实体类型，能为将来导航获得想要的结果，它与 SQL 查询中的 FROM 子句类似：

1：Root 实例是类型化的，且定义了查询的 FROM 子句中能够出现的类型。

2：查询根实例能通过传入一个实体类型给 AbstractQuery.from 方法获得。

3：Criteria 查询，可以有多个查询根。 

4：AbstractQuery 是 CriteriaQuery 接口的父类，它提供得到查询根的方法。CriteriaBuilder 接口：用来构建 CritiaQuery 的构建器对象 Predicate：一个简单或复杂的谓词类型，其实就相当于条件或者是条件组合。 

## 二、**Criteria 查询基本对象的构建**

1：通过 EntityManager 的 getCriteriaBuilder 或 EntityManagerFactory 的 getCriteriaBuilder 方法可以得到 CriteriaBuilder 对象 

2：通过调用 CriteriaBuilder 的 createQuery 或 createTupleQuery 方法可以获得 CriteriaQuery 的实例

3：通过调用 CriteriaQuery 的 from 方法可以获得 Root 实例过滤条件

  A：过滤条件会被应用到 SQL 语句的 FROM 子句中。在 criteria 查询中，查询条件通过 Predicate 或 Expression 实例应用到 CriteriaQuery 对象上。

  B：这些条件使用 CriteriaQuery .where 方法应用到 CriteriaQuery 对象上

  C：CriteriaBuilder 也作为 Predicate 实例的工厂，通过调用 CriteriaBuilder 的条件方（ equal，notEqual， gt， ge，lt， le，between，like 等）创建 Predicate 对象。

  D：复合的 Predicate 语句可以使用 CriteriaBuilder 的 and, or andnot 方法构建。 

​    构建简单的 Predicate 示例：

```java
Predicate p1=cb.like(root.get(“name”).as(String.class), “%”+uqm.getName()+“%”);
Predicate p2=cb.equal(root.get("uuid").as(Integer.class), uqm.getUuid());
Predicate p3=cb.gt(root.get("age").as(Integer.class), uqm.getAge());
```

​    构建组合的 Predicate 示例：

```java
Predicate p = cb.and(p3,cb.or(p1,p2)); 
```

​    当然也可以形如前面动态拼接查询语句的方式，比如：

```java
Specification<UserModel> spec = new Specification<UserModel>() {  
    public Predicate toPredicate(Root<UserModel> root,  
            CriteriaQuery<?> query, CriteriaBuilder cb) {  
        List<Predicate> list = new ArrayList<Predicate>();  
              
        if(um.getName()!=null && um.getName().trim().length()>0){  
            list.add(cb.like(root.get("name").as(String.class), "%"+um.getName()+"%"));  
        }  
        if(um.getUuid()>0){  
            list.add(cb.equal(root.get("uuid").as(Integer.class), um.getUuid()));  
        }  
        Predicate[] p = new Predicate[list.size()];  
        return cb.and(list.toArray(p));  
    }  
};
```

​    也可以使用 CriteriaQuery 来得到最后的 Predicate，示例如下：

```java
Specification<UserModel> spec = new Specification<UserModel>() {  
    public Predicate toPredicate(Root<UserModel> root,  
            CriteriaQuery<?> query, CriteriaBuilder cb) {  
        Predicate p1 = cb.like(root.get("name").as(String.class), "%"+um.getName()+"%");  
        Predicate p2 = cb.equal(root.get("uuid").as(Integer.class), um.getUuid());  
        Predicate p3 = cb.gt(root.get("age").as(Integer.class), um.getAge());  
        //把Predicate应用到CriteriaQuery中去,因为还可以给CriteriaQuery添加其他的功能，比如排序、分组啥的  
        query.where(cb.and(p3,cb.or(p1,p2)));  
        //添加排序的功能  
        query.orderBy(cb.desc(root.get("uuid").as(Integer.class)));  
          
        return query.getRestriction();  
    }  
};
```

综上所述，我们可以用 Specification 的 toPredicate 方法构建更多更复杂的查询方法。甚至如果你想多表链接进行查询，首先在对象上用 @onetoone、@onetomany、@manytoone 等注解，然后利用 root.jion () 方法也能做到，还可以设定链表方式等。具体可以查看相关 jpa 文档。

