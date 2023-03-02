https://blog.csdn.net/qq_44732146/article/details/119968376

https://juejin.cn/post/6923854593512194055

MapStruct是一个Java注释处理器，用于生成类型安全的bean映射类。

定义一个映射器接口，该接口声明任何必需的映射方法。在编译期间，MapStruct将生成此接口的实现。此实现使用简单的Java方法调用在源对象和目标对象之间进行映射，即没有反射或类似内容。与手动编写映射代码相比，MapStruct通过生成繁琐且易于出错的代码来节省时间。遵循配置方法上的约定，MapStruct使用合理的默认值，但在配置或实现特殊行为时不加理会。

与动态映射框架相比，MapStruct具有以下优点：

1. 通过使用普通方法调用（settter/getter）而不是反射来快速执行
2. 编译时类型安全性：只能映射相互映射的对象和属性，不能将order实体意外映射到customer DTO等。
3. 如果有如下问题，编译时会抛出异常
   3.1 映射不完整（并非所有目标属性都被映射）
   3.2 映射不正确（找不到正确的映射方法或类型转换）
4. 可以通过freemarker定制化开发



## 一、基本使用

### 1.Student和StudentDTO转换

```java
@Data
@EqualsAndHashCode(callSuper = false)
@ApiModel(value="Student对象", description="")
public class Student extends Model<Student> {

    private static final long serialVersionUID = 1L;

    @ApiModelProperty(value = "id")
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    @ApiModelProperty(value = "学号")
    private String stuId;

    @ApiModelProperty(value = "名字")
    private String name;

    @ApiModelProperty(value = "性别 0 男，1 女")
    private Integer sex;

    @ApiModelProperty(value = "手机号")
    private String phone;

    @ApiModelProperty(value = "个人简介")
    private String info;

    @ApiModelProperty(value = "头像，picture表id")
    private Long picture;

    @ApiModelProperty(value = "班级")
    private String className;

    @ApiModelProperty(value = "专业")
    private String major;

    @ApiModelProperty(value = "所在小组")
    private Long gId;

}
```

```java
@Data
@EqualsAndHashCode(callSuper = false)
@ApiModel(value = "学生DTO对象", description = "返回某一个学生最基本的信息")
public class StudentDTO {


    @ApiModelProperty(value = "id")
    private Long id;

    @ApiModelProperty(value = "名字")
    private String name;

    @ApiModelProperty(value = "性别 0 男，1 女")
    private Integer sex;

    @ApiModelProperty(value = "班级")
    private String className;

    @ApiModelProperty(value = "学号")
    private Long stuId;

    @ApiModelProperty(value = "手机号")
    private String phone;
}
```

1) 引入依赖

```xml
        <!--Java 实体映射工具 —— mapStruct依赖-->
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct-processor</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>
```



2）**编写对象转换接口**

通常在server层。

![image-20230302222223062](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230302222223062.png)

```java
/**
 * @Mapper 定义这是一个MapStruct对象属性转换接口，在这个类里面规定转换规则
 *          在项目构建时，会自动生成改接口的实现类，这个实现类将实现对象属性值复制。  注解详解看下方。
 */
@Mapper
public interface StudentMapStruct {
    /**
     * 获取该类自动生成的实现类的实例
     * 接口中的属性都是 public static final 的 方法都是public abstract的
     */
    StudentMapStruct INSTANCES = Mappers.getMapper(StudentMapStruct.class);

   //转换对象
    StudentDTO toStudentDTO(Student student);
    //转换集合
    List<StudentDTO> toListStudentDTO(List<Student> student);
}
```

3）使用

```java
	   //设置每页的容量为5，这里是一个分页查询。
        Integer pageCount=2;
        Page<Student> page = new Page<>(pageCurrent,pageCount);
        Page<Student> studentPage = studentMapper.selectPage(page, null);
		//获取数据库查询的对象
        List<Student> students = studentPage.getRecords();
        //转换成DTO集合
        List<StudentDTO> studentsDTO = StudentMapStruct.INSTANCES.toListStudentDTO(students);
        System.out.println(students);
        System.out.println(studentsDTO);
```

### 2.源码分析

创建StudentMapStruct接口后，运行，系统会默认帮我们生成它的实现类。底层是使用最基本的set/get

![image-20230302222432315](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230302222432315.png)

## 二、进阶使用

MapStruct为我们提供了 **@Mapper** 注解，提供了一些属性配置，常用的两种**componentMode**l和**unmappedTargetPolicy**

```java
@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface StudentMapStruct {
}
```

**1.componentModel = "XX"      四个属性值**

- **default:** 默认的情况，mapstruct不使用任何组件类型, 可以通过Mappers.getMapper(Class)方式获取自动生成的实例对象。
- **cdi:** the generated mapper is an application-scoped CDI bean and can be retrieved via @Inject
- **spring（经常使用）:** 生成的实现类上面会自动添加一个@Component注解，可以通过Spring的 @Autowired方式进行注入
- **jsr330:** 生成的实现类上会添加@javax.inject.Named 和@Singleton注解，可以通过 @Inject注解获取。

**2. unmappedTargetPolicy=ReportingPolicy.XX    三个属性值**

![image-20230302222647063](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230302222647063.png)

**- IGNORE** 将被忽略

 **- WARN** 构建时引起警告 

**- ERROR** 映射代码生成失败

#### 配置选项

MapStruct代码生成器有一些处理器选项配置

- mapstruct.suppressGeneratorTimesstamp
  - 如果设置为true，将禁止在生成的映射器类中创建时间戳
  - 默认为false
- mapstruct.suppressGeneratorVersionInfoComment
  - 如果设置为true，将禁止在生成的映射器类中创建属性
  - 默认为false
- mapstruct.defaultComponentModel
  - 基于生成映射器的组件模型的名称
  - 支持：default，cdi，spring，jsr330
  - 默认为default,使用spring可以使用@Autowired方式注入
- mapstruct.unmappedTargetPolicy
  - 在未使用`source`值填充映射方法的`target`的属性的情况下要应用的默认报告策略。
  - 支持：ERROR（映射代码生成失败），WARN（构建时引起警告），IGNORE（将被忽略）
  - 默认为WARN

配置方式如下

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.5.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>${org.mapstruct.version}</version>
            </path>
        </annotationProcessorPaths>
        <compilerArgs>
            <compilerArg>
                -Amapstruct.suppressGeneratorTimestamp=true
            </compilerArg>
            <compilerArg>
                -Amapstruct.suppressGeneratorVersionInfoComment=true
            </compilerArg>
        </compilerArgs>
    </configuration>
</plugin>
```



### 使用Spring依赖

使用了componentModel=“spring”，我们就不需要创建StudentMapStrct的实例了，直接通过 @Autowired即可使用。

```java
接口中：
@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface StudentMapStruct {
    StudentDTO toStudentDTO(Student student);
    List<StudentDTO> toListStudentDTO(List<Student> student);
   }

注入：
    @Autowired  
    StudentMapStruct studentMapStruct;

直接使用：
     StudentDTO studentDTO = studentMapStruct.toStudentDTO(student);
     List<StudentDTO> studentsDTO =studentMapStruct.toListStudentDTO(students);
```

### 属性名称不一致

- 属性不一致时，我们可以使用@Mapping

```java
@Mapping(source = "name", target = "stuName")

@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface StudentMapStruct {
	@Mapping(source = "name", target = "stuName")
    StudentDTO toStudentDTO(Student student);
    @Mapping(source = "name", target = "stuName")
    List<StudentDTO> toListStudentDTO(List<Student> student);
   }
```

### 属性类型不一致

当类型不一致时，mapstruct会帮我们做自动转换，但是自动转换类型有限，**能够自动转换的类型**：

1. 基本类型及其包装类
2. 基本类型的包装类型和String类型之间
3. String类型和枚举类型之间
4. 自定义常量

```java
@Data
public class UserTest {
   private String gender;
   private  Integer age;
}


@Data
public class UserTestDTO {
   private  boolean sex;
   private String age;
}
```

**类型转换类** 要加入到bean容器中，因为之后当我们需要使用时，他会自动的被调用

```java
@Component
public class BooleanStrFormat {
   public String toStr(Boolean gender) {
       if (gender) {
           return "男";
       } else {
           return "女";
       }
   }
   public Boolean toBoolean(String str) {
       if (str.equals("Y")) {
           return true;
       } else {
           return false;
       }
   }
}
```

**映射接口** 使用uses={BooleanStrFormat.class})引入转换类，自动会调用

```java
@Component
@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE,uses={BooleanStrFormat.class})
public interface UserStruct {
    @Mappings({
            @Mapping(source = "gender",target = "sex"),
    })
    UserTestDTO toUserTestDTO(UserTest userTest);
}
```

### 多个source映射一个target

能会遇到将两个不同的对象的值，赋给同一个对象，此时我们就需要使用@Mapping这个注解。

```java
@Mapper(componentModel = "spring")
public interface GoodInfoMapper {
    @Mappings({
            @Mapping(source = "type.name",target = "typeName"),
            @Mapping(source = "good.id",target = "goodId"),
            @Mapping(source = "good.title",target = "goodName"),
            @Mapping(source = "good.price",target = "goodPrice")
    })
    GoodInfoDTO fromGoodInfoDTO(GoodInfo good, GoodType type);
}
```

### 日期转换/固定值

```java
 固定值：constant
    @Mapping(source = "name", constant = "hollis")
  日期转换：dateForma
    @Mapping(source = "person.begin",target="birth" dateFormat="yyyy-MM-dd HH:mm:ss")
```

### 映射对象引用

通常一个对象不仅具有基本属性，而且还引用了其他对象
例如：Car类包含一个Person对象，该对应应该映射到CarDto中的PersionDto.
这种情况下，只需为引用的对象类型定义一个映射方法。

```java
@Mapper
public interface CarMapper {
    CarDto carToCarDto(Car car);
    PersonDto personToPersonDto(Person person);
}
```

生成映射方法是，MapStruct将为`source`对象和`target`中的每个属性对应以下规则：

1. 如果source和target属性具有相同的类型，则简单的将source复制到target，如果属性是一个集合，则该集合的副本将设置到`target`属性中，集合类型生成代码如下：

```java
	List<Integer> list = s.getListTest();
    if ( list != null ) {
        target.setListTest( new ArrayList<Integer>( list ) );
    }
```

1. 如果`source`属性和`target`属性类型不同，检查是否存在另一种映射方式，该映射方法将`source`属性的类型作为参数类型并将`target`属性的类型作为返回类型。如果存在这样的方法，它将在生成的映射实现中调用。
2. 如果不存在这样的方法，MapStruct将查看是否存在属性的`source`类型和`target`类型的内置转换。在这种情况下，生成的映射代码将应用此转换。
3. 如果找不到这种方法，MapStruct将尝试生成自动子映射方法，该方法将在`source`属性和`target`属性之间进行映射。
4. 如果MapStruct无法创建基于名称的映射方法，则会在构建时引发错误，指示不可映射的属性及其路径。



### 隐式类型转换

MapStruct会自动处理类型转换，例如：如果某个属性int在souce中类型为String，则生成的代码将分别通过调用String.valueOf(int),和Integer.parseInt(String)执行转换。

以下转换将自动应用:

- 在所有Java原语数据类型及其对应的包装器类型之间（例如在int和之间Integer，boolean以及Boolean等）之间。生成的代码是已知的null，即，当将包装器类型转换为相应的原语类型时，null将执行检查.
- 在所有Java数字类型和包装器类型之间，例如在int和long或byte和之间Integer。

> 从较大的数据类型转换为较小的数据类型（例如从long到int）可能会导致值或精度损失。在Mapper和MapperConfig注释有一个方法typeConversionPolicy来控制警告/错误。由于向后兼容的原因，默认值为“ ReportingPolicy.IGNORE”。

- 所有Java基本类型之间（包括其包装）和String之间，例如int和String或Boolean和String。java.text.DecimalFormat可以指定理解的格式字符串。

```java
@IterableMapping(numberFormat = "$#.00")
List<String> prices(List<Integer> prices);
```

自动生成的映射类方法:

```java
@Override
public List<String> prices(List<Integer> prices) {
    if ( prices == null ) {
        return null;
    }

    List<String> list = new ArrayList<String>( prices.size() );
    for ( Integer integer : prices ) {
        list.add( new DecimalFormat( "$#.00" ).format( integer ) );
    }

    return list;
}
```

- 在enum类型和之间String。
- 在大数字类型（java.math.BigInteger，java.math.BigDecimal）和Java基本类型（包括其包装器）以及字符串之间。java.text.DecimalFormat可以指定理解的格式字符串
- 在java.util.Date/ XMLGregorianCalendar和之间String。java.text.SimpleDateFormat可以通过以下dateFormat选项指定可理解的格式字符串



### 默认值和常量

如果相应的source属性为null，则可以指定将`target`的属性设置为默认值。在任何情况下都可以指定常量来设置这样的预定义值。
默认值和常量使用String来指定，当`target`类型是基础类型或者包装类时，映射类会进行自动的类型转换，以匹配`target`属性所需的类型.

```java
    @Mapping(target = "name", constant = "常量")
    @Mapping(target = "quantity", defaultValue = "1L")
    OrderItem toOrder(OrderItemDto orderItemDto);
```

```java
	@Override
    public OrderItem toOrder(OrderItemDto orderItemDto) {
        if ( orderItemDto == null ) {
            return null;
        }

        OrderItem orderItem = new OrderItem();

        if ( orderItemDto.quantity != null ) {
            orderItem.setQuantity( orderItemDto.quantity );
        }
        else {
            orderItem.setQuantity( (long) 1L );
        }

        orderItem.setName( "常量" );

        return orderItem;
    }
```



### 表达式

该功能对于调用函数很有用，整个source对象在表达式中都是可被调用的。MapStruct不会生成验证表达式，但在编译过程中错误会显示在生成的类中。

```java
@Mapper( imports = TimeAndFormat.class )
public interface SourceTargetMapper {

    SourceTargetMapper INSTANCE = Mappers.getMapper( SourceTargetMapper.class );

    @Mapping(target = "timeAndFormat",
         expression = "java( new TimeAndFormat( s.getTime(), s.getFormat() ) )")
    Target sourceToTarget(Source s);
}
```

MapStruct不会处理类的导入，但是可以在`@Mapper`注解中使用`imports`中引入.

默认表达式是默认值和表达式的组合，当source属性为null是才使用:

```java
@Mapper( imports = UUID.class )
public interface SourceTargetMapper {

    SourceTargetMapper INSTANCE = Mappers.getMapper( SourceTargetMapper.class );

    @Mapping(target="id", source="sourceId", defaultExpression = "java( UUID.randomUUID().toString() )")
    Target sourceToTarget(Source s);
}
```



### 配置的继承

方法级的配置注解，例如：`@Mapping`, `@BeanMapping`, `@IterableMapping`等等，都可以使用注解`@InheritConfiguration`从一个映射方法的类似使用中继承:

```java
@Mapper
public interface CarMapper {

    @Mapping(target = "numberOfSeats", source = "seatCount")
    Car carDtoToCar(CarDto car);

    @InheritConfiguration
    void carDtoIntoCar(CarDto carDto, @MappingTarget Car car);
}
```

上面的示例中声明了`carDtoToCar()`具有配置的映射方法，在进行`updateCar`的时候使用注解`@InheritConfiguration`MapStruct就可以搜索可以继承的方法进行映射。
如果存在多个可以用来继承的方法的时候，就需要在当前的映射器中定义需要继承的方法：`@InheritConfiguration( name = "carDtoToCar" )`

### 逆映射

在双向映射的情况下，正向方法和返现方法的映射规则通常都是相似的，就可以简单的切换source和targe加上继承的注解，进行逆映射。

```java
    @Mapping(source = "orders", target = "orderItems")
    @Mapping(source = "customerName", target = "name")
    Customer toCustomer(CustomerDto customerDto);

    @InheritInverseConfiguration
    CustomerDto fromCustomer(Customer customer);
```



## 三、封装使用

封装一个struct的类，来满足所有对象对映射的基本使用：

**1.创建公共的处理类BaseMapper**

```java
public interface BaseMapper<D, E> {

    /**
     * DTO转Entity
     * @param dto /
     * @return /
     */
    E toEntity(D dto);

    /**
     * Entity转DTO
     * @param entity /
     * @return /
     */
    D toDto(E entity);

    /**
     * DTO集合转Entity集合
     * @param dtoList /
     * @return /
     */
    List <E> toEntity(List<D> dtoList);

    /**
     * Entity集合转DTO集合
     * @param entityList /
     * @return /
     */
    List <D> toDto(List<E> entityList);
}
```

**2.创建每一个实体对象的struct，继承BaseMapper**

```java
@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface StudentMapStruct extends BaseMapper<StudentDTO,Student> {
}
```

**3.使用**

```java
	    StudentDTO studentDTO = studentMapStruct.toDto(student);
        
        List<StudentDTO> studentsDTO =studentMapStruct.toDto(students);
```

## 四、注解介绍

```java
@Mapper 在接口上添加这个注解，MapStruct才会去实现该接口
    using: 应用外部映射类
    import： 导入表达式使用的类
    componentModel: 指定实现类的类型
        default：默认，通过Mappers.getMapper(Class)方式获取实例对象
        spring: 自动添加注解@Component，通过@Autowired方式注入
@Mapping: 属性映射，若source对象与target对象名称一致，会自动映射对应属性
    source： 源属性
    target： 目标属性
    deteFormat： String 到Date日期之间相互转换，通过SimpleDateFormat
    ignore: 忽略这个字段
@Mappings：配置多个@Mapping    
@MappingTarget： 用于更新已有对象
@InheritConfiguration： 用于继承配置
```

