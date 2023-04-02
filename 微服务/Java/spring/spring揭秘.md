## 一、IOC基本概念

Inversion of Control，中文通常翻译为“控制反转”，它还有一个别名叫做依赖注入（ Dependency Injection）。  

如果我们依赖于某个类或服务，最简单而有效的方式就是直接在类的构造函数中新建相应的依赖类。每次用到什么依赖对象都要主动地去获取，这是否真的必要？最终所要做的，其实就是直接调用依赖对象所提供的某项服务而已。只要用到这个依赖对象的时候，它能够准备就绪，我们完全可以不管这个对象是自己找来的还是别人送过来的。 IoC就是为了帮助我们避免之前的“大费周折”，而提供了更加轻松简洁的方式。它的反转，就反转在让你从原来的事必躬亲，转变为现在的享受服务。  

通常情况下，被注入对象会直接依赖于被依赖对象。但是， 在IoC的场景中，二者之间通过IoC Service Provider来打交道，所有的被注入对象和依赖对象现在由IoC Service Provider统一管理。被注入对象需要什么，直接跟IoC Service Provider招呼一声，后者就会把相应的被依赖对象注入到被注入对象中，从而达到IoC Service Provider为被注入对象服务的目的。 IoC Service Provider在这里就是通常的IoC容器所充当的角色。从被注入对象的角度看，与之前直接寻求依赖对象相比，依赖对象的取得方式发生了反转，控制也从被注入对象转到了IoC Service Provider那里。

### 1.依赖注入方式

#### 1.构造方法注入

构造方法注入，就是被注入对象可以通过在其构造方法中声明依赖对象的参数列表， 8让外部（通常是IoC容器）知道它需要哪些依赖对象。  

```java
public FXNewsProvider(IFXNewsListener newsListner,IFXNewsPersister newsPersister) 10
{
    this.newsListener = newsListner;
    this.newPersistener = newsPersister;
}
```

IoC Service Provider会检查被注入对象的构造方法，取得它所需要的依赖对象列表，进而为其注入相应的对象。**同一个对象是不可能被构造两次的**，因此，被注入对象的构造乃至其整个生命周期，应该是由IoC Service Provider来管理的。  

对象在构造完成之后，即已进入就绪状态，可以马上使用。缺点就是，当依赖对象比较多的时候，构造方法的参数列表会比较长。而通过反射构造对象的时候，对相同类型的参数的处理会比较困难，维护和使用上也比较麻烦。  

#### 2.setter方法注入

当前对象只要为其依赖对象所对应的属性添加setter方法，就可以通过setter方法将相应的依赖对象设置到被注入对象中。  

```java
public void setNewsListener(IFXNewsListener newsListener) {
	this.newsListener = newsListener;
}
```

setter方法注入虽不像构造方法注入那样，让对象构造完成后即可使用，但相对来说更宽松一些，可以在对象构造完成后再注入。  

setter方法可以被继承，允许设置默认值，而且有良好的IDE支持。缺点当然就是对象无法在构造完成后马上进入就绪状态。

#### 3.接口注入

被注入对象如果想要IoC Service Provider为其注入依赖对象，就必须实现某个接口。这个接口提供一个方法，用来为其注入依赖对象。IoC Service Provider最终通过这些接口来了解应该为被注入对象注入什么依赖对象。  

FXNewsProvider为了让IoC Service Provider为其注入所依赖的IFXNewsListener，首先需要实现IFXNewsListenerCallable接口，这个接口会声明一个injectNewsListner方法（方法名随意），该方法的参数，就是所依赖对象的类型。这样， InjectionServiceContainer对象，即对应的IoC Service Provider就可以通过这个接口方法将依赖对象注入到被注入对象FXNewsProvider当中。  

![image-20230402214400765](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230402214400765.png)

从注入方式的使用上来说，接口注入是现在不甚提倡的一种方式。



IoC是一种可以帮助我们解耦各业务对象间依赖关系的对象绑定方式！

## 二、Ioc Service Provider

 最终需要通过某种角色或者服务将这些相互依赖的对象绑定到一起，而IoC Service Provider就对应IoC场景中的这一角色，可以指代任何将IoC场景中的业务对象绑定到一起的实现方式。Spring的IoC容器就是一个提供依赖注入服务的IoC Service Provider。  

### 1.Ioc Service Provider职责

主要有两个：业务对象的构建管理和业务对象间的依赖绑定  

- 业务对象的构建管理。在IoC场景中，业务对象无需关心所依赖的对象如何构建如何取得，但这部分工作始终需要有人来做。所以， IoC Service Provider需要将对象的构建逻辑从客户端对象那里剥离出来，以免这部分逻辑污染业务对象的实现。  
- 业务对象间的依赖绑定。对于IoC Service Provider来说，这是它的最终使命之所在。如果不能完成这个职责， IoC Service Provider通过结合之前构建和管理的所有业务对象，以及各个业务对象间可以识别的依赖关系，将这些对象所依赖的对象注入绑定，从而保证每个业务对象在使用的时候，可以处于就绪状态。  

### 2.IoC Service Provider 如何管理对象间的依赖关系  

#### 1.直接编码方式

当前大部分的IoC容器都应该支持直接编码方式，在容器启动之前，我们就可以通过程序编码的方式将被注入对象和依赖对象注册到容器中，并明确它们相互之间的依赖注入关系。  

```java
IoContainer container = ...;
container.register(FXNewsProvider.class,new FXNewsProvider());
container.register(IFXNewsListener.class,new DowJonesNewsListener());
...
FXNewsProvider newsProvider = (FXNewsProvider)container.get(FXNewsProvider.class);
newProvider.getAndPersistNews();
```



#### 2.配置文件方式

一种较为普遍的依赖注入关系管理方式。像普通文本文件、 properties文件、 XML文件等，都可以成为管理依赖注入关系的载体。最为常见的，还是通过XML文件来管理对象注册和对象间依赖关系。

```xml
<bean id="newsProvider" class="..FXNewsProvider">
<property name="newsListener">
<ref bean="djNewsListener"/>
</property>
<property name="newPersistener">
<ref bean="djNewsPersister"/>
</property>
</bean>
<bean id="djNewsListener"
class="..impl.DowJonesNewsListener">
</bean>
<bean id="djNewsPersister"
class="..impl.DowJonesNewsPersister">
</bean>
```

```java
container.readConfigurationFiles(...);
FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("newsProvider");
newsProvider.getAndPersistNews();
```



#### 3.元数据方式

这种方式的代表实现是Google Guice。



## 三、BeanFactory

Spring的IoC容器是一个提供IoC支持的轻量级容器，除了基本的IoC支持，它作为轻量级容器还提供了IoC之外的支持。  

![image-20230402215435208](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230402215435208.png)

Spring提供了两种容器类型： **BeanFactory和ApplicationContext**：

- BeanFactory。基础类型IoC容器，提供完整的IoC服务支持。如果没有特殊指定，默认采用延迟初始化策略（ lazy-load）。只有当客户端对象需要访问容器中的某个受管对象的时候，才对该受管对象进行初始化以及依赖注入操作。所以，相对来说，容器启动初期速度较快，所需要的资源有限。对于资源有限，并且功能要求不是很严格的场景， BeanFactory是比较合适的IoC容器选择。
- ApplicationContext。 ApplicationContext在BeanFactory的基础上构建，是相对比较高级的容器实现，除了拥有BeanFactory的所有支持， ApplicationContext还提供了其他高级特性，比如事件发布、国际化信息支持等。ApplicationContext所管理的对象，在该类型容器启动之后，默认全部初始化并绑定完成。所以，相对于BeanFactory来说， ApplicationContext要求更多的系统资源，同时，因为在启动时就完成所有初始化，容器启动时间较之BeanFactory也会长一些。在那些系统资源充足，并且要求更多功能的场景中，ApplicationContext类型的容器是比较合适的选择。  

![image-20230402215625036](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230402215625036.png)

ApplicationContext间接继承自BeanFactory，所以说它是构建于BeanFactory之上的IoC容器。  

### 1.BeanFactory的对象注册与依赖绑定方式  

#### 1.直接编码方式  

BeanFactory只是一个接口，最终需要一个该接口的实现来进行实际的Bean的管理， DefaultListableBeanFactory就是这么一个比较通用的BeanFactory实现类。 DefaultListableBeanFactory除了间接地实现了BeanFactory接口，还实现了BeanDefinitionRegistry接口，该接口才是在BeanFactory的实现中担当Bean注册管理的角色。BeanFactory接口只定义如何访问容器内管理的Bean的方法，各个BeanFactory的具体实现类负责具体Bean的注册以及管理工作。  BeanDefinitionRegistry接口定义抽象了Bean的注册逻辑。  

![image-20230402220032059](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230402220032059.png)

每一个受管的对象，在容器中都会有一个BeanDefinition的实例（ instance）与之相对应，该BeanDefinition的实例负责保存对象的所有必要信息，包括其对应的对象的class类型、是否是抽象类、构造方法参数以及其他属性等。当客户端向BeanFactory请求相应对象的时候， BeanFactory会通 过 这 些 信 息 为 客 户 端 返 回 一 个 完 备 可 用 的 对 象 实 例 。 RootBeanDefinition和 ChildBeanDefinition是BeanDefinition的两个主要实现类。  

```java
public static void main(String[] args)
{
    DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
    BeanFactory container = (BeanFactory)bindViaCode(beanRegistry);
    FXNewsProvider newsProvider = ➥
    (FXNewsProvider)container.getBean("djNewsProvider");
    newsProvider.getAndPersistNews();
}
public static BeanFactory bindViaCode(BeanDefinitionRegistry registry)
{
    AbstractBeanDefinition newsProvider = ➥
    new RootBeanDefinition(FXNewsProvider.class,true);
    AbstractBeanDefinition newsListener = ➥
    new RootBeanDefinition(DowJonesNewsListener.class,true);
    AbstractBeanDefinition newsPersister = ➥
    new RootBeanDefinition(DowJonesNewsPersister.class,true);
    // 将bean定义注册到容器中
    registry.registerBeanDefinition("djNewsProvider", newsProvider);
    registry.registerBeanDefinition("djListener", newsListener);
    registry.registerBeanDefinition("djPersister", newsPersister);
    // 指定依赖关系
    // 1. 可以通过构造方法注入方式
    ConstructorArgumentValues argValues = new ConstructorArgumentValues();
    argValues.addIndexedArgumentValue(0, newsListener);
    argValues.addIndexedArgumentValue(1, newsPersister);
    newsProvider.setConstructorArgumentValues(argValues);
    // 2. 或者通过setter方法注入方式
    MutablePropertyValues propertyValues = new MutablePropertyValues();
    propertyValues.addPropertyValue(new ropertyValue("newsListener",newsListener));
    propertyValues.addPropertyValue(new PropertyValue("newPersistener",newsPersister));
    newsProvider.setPropertyValues(propertyValues);
    // 绑定完成 2
    return (BeanFactory)registry;
}
```

在 main 方 法 中 ， 首 先 构 造 一 个 DefaultListableBeanFactory 作 为 BeanDefinitionRegistry，然后将其交给bindViaCode方法进行具体的对象注册和相关依赖管理，然后通过bindViaCode返回的BeanFactory取得需要的对象，最后执行相应逻辑。  

在bindViaCode方法中，首先针对相应的业务对象构造与其相对应的BeanDefinition，使用了 RootBeanDefinition 作 为 BeanDefinition 的 实 现 类 。 构 造 完 成 后 ， 将 这 些BeanDefinition注册到通过方法参数传进来的BeanDefinitionRegistry中。之后，因为FXNewsProvider是采用的构造方法注入，所以，需要通过ConstructorArgumentValues为其注入相关依赖。在这里为了同时说明setter方法注入，也同时展示了在Spring中如何使用代码实现setter方法注入。如果要运行这段代码，需要把setter方法注入部分的4行代码注释掉。最后，以BeanFactory的形式返回已经注册并绑定了所有相关业务对象的BeanDefinitionRegistry实例。  

#### 2.外部配置文件方式  

Spring的IoC容器支持两种配置文件格式： Properties文件格式和XML文件格式。如果你愿意也可以引入自己的文件格式。

采用外部配置文件时， Spring的IoC容器有一个统一的处理方式。通常情况下，需要根据不同的外部配置文件格式，给出相应的BeanDefinitionReader实现类，由BeanDefinitionReader的相应实现类负责将相应的配置文件内容读取并映射到BeanDefinition，然后将映射后的BeanDefinition注册到一个BeanDefinitionRegistry，之后， BeanDefinitionRegistry即完成Bean的注册和加载。  

大部分工作，包括解析文件格式、装配BeanDefinition之类的工作，都是由BeanDefinitionReader的相应实现来做的， BeanDefinitionRegistry只不过负责保管而已。  

```java
BeanDefinitionRegistry beanRegistry = <某个BeanDefinitionRegistry实现类，通常为➥
DefaultListableBeanFactory>;
BeanDefinitionReader beanDefinitionReader = new BeanDefinitionReaderImpl(beanRegistry);
beanDefinitionReader.loadBeanDefinitions("配置文件路径");
// 现在我们就取得了一个可用的BeanDefinitionRegistry实例
```

##### 1.Properties配置格式的加载  

Spring提供了org.springframework.beans.factory.support.PropertiesBeanDefinitionReader类用于Properties格式配置文件的加载，所以，不用自己去实现BeanDefinitionReader，只要根据该类的读取规则，提供相应的配置文件即可。  

Spring提供的PropertiesBeanDefinitionReader是按照Spring自己的文件配置规则进行加载的，而同样的道理，也可以按照自己的规则 来提供相应的Properties配置文件。只不过，现在需要实现你自己的“PropertiesBeanDefinitionReader”来读取并解析。  

这种方式在后面版本被废弃。

##### 2.XML配置格式的加载  

XML配置格式是Spring支持最完整，功能最强大的表达方式。  

```java
public static void main(String[] args)
{
    DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
    BeanFactory container = (BeanFactory)bindViaXMLFile(beanRegistry);
    FXNewsProvider newsProvider = ➥
    (FXNewsProvider)container.getBean("djNewsProvider");
    newsProvider.getAndPersistNews(); 2
}
public static BeanFactory bindViaXMLFile(BeanDefinitionRegistry registry)
    { 3
    XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(registry);
    reader.loadBeanDefinitions("classpath:../news-config.xml");
    return // 或者直接 (BeanFactory)registry; 4
    //return new XmlBeanFactory(new ClassPathResource("../news-config.xml"));
}
```

Spring同样为XML格 式 的 配 置 文 件 提 供 了 现 成 的 BeanDefinitionReader实 现 ， 即 XmlBeanDefinitionReader。XmlBeanDefinitionReader负责读取Spring指定格式的XML配置文件并解析，之后将解析后的文件内容映射到相应的BeanDefinition，并加载到相应的BeanDefinitionRegistry中（在这里是DefaultListableBeanFactory）。这时，整个BeanFactory就可以放给客户端使用了。  

如果出于某种目的， XmlBeanFactory或者默认的XmlBeanDefinitionReader所使用的XML格式无法满足需要的话，你同样可以通过扩展XmlBeanDefinitionReader或者直接实现自己的BeanDefinitionReader来达到自定义XML配置文件加载的目的。   

#### 3.注解方式

如果要通过注解标注的方式为注入所需要的依赖，现在可以使用@Autowired以及@Component对相关类进行标记。  @Autowired是这里的主角，它的存在将告知Spring容器需要为当前对象注入哪些依赖对象。而@Component则是配合Spring 2.5中新的classpath-scanning功能使用的。  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" ➥
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ➥
xmlns:context="http://www.springframework.org/schema/context" ➥
xmlns:tx="http://www.springframework.org/schema/tx" ➥
xsi:schemaLocation="http://www.springframework.org/schema/beans ➥
http://www.springframework.org/schema/beans/spring-beans-2.5.xsd ➥
http://www.springframework.org/schema/context ➥
http://www.springframework.org/schema/context/spring-context-2.5.xsd ➥
http://www.springframework.org/schema/tx ➥
http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">
<context:component-scan base-package="cn.spring21.project.base.package"/>
</beans>
```

<context:component-scan/>会到指定的包（ package）下面扫描标注有@Component的类，如果找到，则将它们添加到容器进行管理，并根据它们所标注的@Autowired为这些类注入符合条件的依赖对象。  

```java
public static void main(String[] args)
{
    ApplicationContext ctx = new ClassPathXmlApplicationContext("配置文件路径");
    FXNewsProvider newsProvider = (FXNewsProvider)container.getBean("FXNewsProvider");
    newsProvider.getAndPersistNews();
}
```

### 2.Spring的XML

#### 1.<beans>和<bean>  



## 四、深入Ioc容器

Spring的IoC容器所起的作用 ，会以某种方式加载Configuration Metadata（通常也就是XML格式的配置信息），然后根据这些信息绑定整个系统的对象，最终组装成一个可用的基于轻量级容器的应用系统。  

Spring的IoC容器实现以上功能的过程，基本上可以按照类似的流程划分为两个阶段，即容器启动 8
阶段和Bean实例化阶段 。Spring的IoC容器在实现的时候，充分运用了这两个实现阶段的不同特点，在每个阶段都加入了相应的容器扩展点，以便我们可以根据具体场景的需要加入自定义的扩展逻辑。  

![image-20230402221427577](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230402221427577.png)

容器启动伊始，首先会通过某种途径加载Configuration MetaData。除了代码方式比较直接，在大部分情况下，容器需要依赖某些工具类（ BeanDefinitionReader）对加载的Configuration MetaData进行解析和分析，并将分析后的信息编组为相应的BeanDefinition，最后把这些保存了bean定义必要信息的BeanDefinition，注册到相应的BeanDefinitionRegistry，这样容器启动工作就完成了。   

![image-20230402221510476](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230402221510476.png)

经过第一阶段，现在所有的bean定义信息都通过BeanDefinition的方式注册到了BeanDefinitionRegistry中。当某个请求方通过容器的getBean方法明确地请求某个对象，或者因依赖关系容器需要隐式地调用getBean方法时，就会触发第二阶段的活动。  

该阶段，容器会首先检查所请求的对象之前是否已经初始化。如果没有，则会根据注册的BeanDefinition所提供的信息实例化被请求对象，并为其注入依赖。如果该对象实现了某些回调接口，也会根据回调接口的要求来装配它。当该对象装配完毕之后，容器会立即将其返回请求方使用。如果说第一阶段只是根据图纸装配生产线的话，那么第二阶段就是使用装配好的生产线来生产具体的产品了。  

Spring提供了一种叫做BeanFactoryPostProcessor的容器扩展机制。该机制允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做相应的修改。这就相当于在容器实现的第一阶段最后加入一道工序，让我们对最终的BeanDefinition做一些额外的操作，比如修改其中bean定义的某些属性，为bean定义增加其他信息等。  

如果要自定义实现BeanFactoryPostProcessor，通常我们需要实现org.springframework.beans.factory.config.BeanFactoryPostProcessor接口。同时，因为一个容器可能拥有多个BeanFactoryPostProcessor，这个时候可能需要实现类同时实现Spring的org.springframework.core.Ordered接口，以保证各个BeanFactoryPostProcessor可以按照预先设定的顺序执行（如果顺序紧
要的话）。

但是，因为Spring已经提供了几个现成的BeanFactoryPostProcessor实现类，所以，大多时候，我们很少自己去实现某个BeanFactoryPostProcessor。其中， org.springframework.beans.factory.config.PropertyPlaceholderConfigurer和org.springframework.beans.factory.config.Property OverrideConfigurer是两个比较常用的BeanFactoryPostProcessor。

另外，为了处理配置文件中的数据类型与真正的业务对象所定义的数据类型转换， Spring还允许我们通过org.springframework.beans.factory.config.CustomEditorConfigurer来注册自定义的PropertyEditor以补助容器中默认的PropertyEditor。  

通 过 两 种 方 式 来 应 用 BeanFactoryPostProcessor ， 分 别 针 对 基 本 的 IoC 容 器BeanFactory和较为先进的容器ApplicationContext。  

```java
// 声明将被后处理的BeanFactory实例
ConfigurableListableBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("..."));
// 声明要使用的BeanFactoryPostProcessor 3
PropertyPlaceholderConfigurer propertyPostProcessor = new PropertyPlaceholderConfigurer();
propertyPostProcessor.setLocation(new ClassPathResource("..."));
// 执行后处理操作
propertyPostProcessor.postProcessBeanFactory(beanFactory); 
```

对于ApplicationContext来说，情况看起来要好得多。因为ApplicationContext会自动识别配置文件中的BeanFactoryPostProcessor并应用它，所以， 相对于BeanFactory，在ApplicationContext中加载并应用BeanFactoryPostProcessor，仅需要在XML配置文件中将这些BeanFactoryPostProcessor简单配置一下即可。  

```xml
<beans>
    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations"> 
            <list>
                <value>conf/jdbc.properties</value>
                <value>conf/mail.properties</value>
            </list> 
        </property>
    </bean>
    ... 
</beans>
```

- PropertyPlaceholderConfigurer  ：允许我们在XML配置文件中使用占位符（ PlaceHolder），并将这些占位符所代表的资源单独配置到简单的properties文件中来加载。  

  当BeanFactory在第一阶段加载完成所有配置信息时， BeanFactory中保存的对象的属性信息还只是以占位符的形式存在，如${jdbc.url}、 ${jdbc.driver}。当PropertyPlaceholderConfigurer作为BeanFactoryPostProcessor被应用时，它会使用properties配置文件中的配置信息来替换相应BeanDefinition中占位符所表示的属性值。这样，当进入容器实现的第二阶段实例化bean时， bean定义中的属性值就是最终替换完成的了。  

- PropertyOverrideConfigurer ：PropertyOverrideConfigurer对容器中配置的任何你想处理的bean定义的property信息进行覆盖替换。  

- CustomEditorConfigurer ：其他两个BeanFactoryPostProcessor都是通过对BeanDefinition中的数据进行变更以达到某种目的。  CustomEditorConfigurer只是辅助性地将后期会用到的信息注册到容器，对BeanDefinition没有做任何变动。  容器从XML格式的文件中读取的都是字符串形式，最终应用程序却是由各种类型的对象所构成。要想完成这种由字符串到具体对象的转换（不管这个转换工作最终由谁来做），都需要这种转换规则相关的信息，而CustomEditorConfigurer就是帮助我们传达类似信息的。  

在已经可以借助于**BeanFactoryPostProcessor来干预Magic实现的第一个阶段（容器启动阶段）**之后，下一个阶段，即bean实例化阶段。

容器现在仅仅拥有所有对象的BeanDefinition来保存实例化阶段将要用的必要信息。只有当请求方通过BeanFactory的getBean()方法来请求某个对象实例的时候，才有可能触发Bean实例化阶段的活动。 BeanFactory的getBe法可以被客户端对象显式调用，也可以在容器内部隐式地被调用。隐式调用有如下两种情况。 ：

- 对于BeanFactory来说，对象实例化默认采用延迟初始化。通常情况下，当对象A被请求而需要第一次实例化的时候，如果它所依赖的对象B之前同样没有被实例化，那么容器会先实例化对象A所依赖的对象。这时容器内部就会首先实例化对象B，以及对象 A依赖的其他还没有实例化的对象。这种情况是容器内部调用getBean()，对于本次请求的请求方是隐式的。
- ApplicationContext启动之后会实例化所有的bean定义，但ApplicationContext在实现的过程中依然遵循Spring容器实现流程的两个阶段，只不过它会在启动阶段的活动完成之后，紧接着调用注册到该容器的所有bean定义的实例化方法getBean()。这就是为什么当你得到ApplicationContext类型的容器引用时，容器内所有对象已经被全部实例化完成。不信你查一下类org.AbstractApplicationContext的refresh()方法。  

之所以说getBean()方法是有可能触发Bean实例化阶段的活动，是因为只有当对应某个bean定义的getBean()方法第一次被调用时，不管是显式的还是隐式的， Bean实例化阶段的活动才会被触发，第二次被调用则会直接返回容器缓存的第一次实例化完的对象实例（ prototype类型bean除外）。当getBean()方法内部发现该bean定义之前还没有被实例化之后， 会通过createBean()方法来进行具体的对象实例化 ：

![image-20230402222631185](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230402222631185.png)

Spring容器将对其所管理的对象全部给予统一的生命周期管理，这些被管理的对象完全摆脱了原来那种“ new完后被使用，脱离作用域后即被回收”的命运。  

### 1.每个bean在容器中是如何走过其一生的  
#### 1.Bean的实例化与BeanWrapper  



#### 2.各色的Aware接口 



#### 3.BeanPostProcessor  



#### 4.InitializingBean和init-method  



#### 5.DisposableBean与destroy-method  