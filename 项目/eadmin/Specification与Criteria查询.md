https://www.freesion.com/article/6425514764/

## 一、**Criteria 查询基本概念**

Spring Data JPA 支持 JPA2.0 的 Criteria 查询，相应的接口是 JpaSpecificationExecutor。Criteria 查询：是一种类型安全和更面向对象的查询 。这个接口基本是围绕着 Specification 接口来定义的， Specification 接口中只定义了如下一个方法：

```
Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb); 
```

Criteria 查询是以元模型的概念为基础的，元模型是为具体持久化单元的受管实体定义的，这些实体可以是实体类，嵌入类或者映射的父类。

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

```
Predicate p1=cb.like(root.get(“name”).as(String.class), “%”+uqm.getName()+“%”);
Predicate p2=cb.equal(root.get("uuid").as(Integer.class), uqm.getUuid());
Predicate p3=cb.gt(root.get("age").as(Integer.class), uqm.getAge());
```

​    构建组合的 Predicate 示例：

```
Predicate p = cb.and(p3,cb.or(p1,p2)); 
```

​    当然也可以形如前面动态拼接查询语句的方式，比如：

```
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

```
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

