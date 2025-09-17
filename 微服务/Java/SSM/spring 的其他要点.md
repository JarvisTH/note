## spring boot部署与监控

https://jiagouyizhan.com/course/1721806067404378113

## 高级开发架构面试

https://jiagouyizhan.com/course/1721804745338785793

## 微服务架构设计

https://jiagouyizhan.com/course/1721814391243935746

## 一、spring boot 初始化器initiallizer

https://www.jianshu.com/p/0a2112ea318f

## 二、spring boot的监听器

https://www.jianshu.com/p/d5b941dcc663

## 三、spring boot的启动加载器

https://www.jianshu.com/p/071547dafa2b

## 四、spring 的 Aware

https://www.jianshu.com/p/7581b7682cbe

## 五、spring 的Environment

https://www.jianshu.com/p/7581b7682cbe

## 六、spring的profile

### 1、作用

- 将不同的配置参数绑定在不同的环境

### 2.properties 配置

假设，一个应用的工作环境有：dev、test、prod
 那么，我们可以添加 4 个配置文件：

- applcation.properties - 公共配置
- application-dev.properties - 开发环境配置
- application-test.properties - 测试环境配置
- application-prod.properties - 生产环境配置

在 applcation.properties 文件中可以通过以下配置来激活 profile：

```
spring.profiles.active = test
```

### 3.yml 配置

与 properties 文件类似，我们也可以添加 4 个配置文件：

- applcation.yml - 公共配置
- application-dev.yml - 开发环境配置
- application-test.yml - 测试环境配置
- application-prod.yml - 生产环境配置

在 applcation.yml 文件中可以通过以下配置来激活 profile：

```
spring:
  profiles:
    active: prod
```

yml 文件也可以在一个文件中完成所有 profile 的配置：

```yaml
# 激活 prod
spring:
  profiles:
    active: prod
# 也可以同时激活多个 profile
# spring.profiles.active: prod,proddb,prodlog
---
# dev 配置
spring:
  profiles: dev

# 略去配置

---
spring:
  profiles: test

# 略去配置

---
spring.profiles: prod
spring.profiles.include:
  - proddb
  - prodlog

---
spring:
  profiles: proddb

# 略去配置

---
spring:
  profiles: prodlog
# 略去配置
```

不同 profile 之间通过`---`分割

### 4.激活 profile

- ##### 插件激活 profile

```
spring-boot:run -Drun.profiles=prod
```

- ##### VM options、Program arguments、Active Profile

VM options设置启动参数：`-Dspring.profiles.active=prod`
 Program arguments设置：`--spring.profiles.active=prod`
 Active Profile 设置 prod
 这三个参数不要一起设置，会引起冲突，选一种即可，如下图

![image-20240315134210538](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20240315134210538.png)

- ##### 命令行

将项目打成jar包，在jar包的目录下打开命令行，使用如下命令启动：

```cmd
java -jar spring-boot-profile.jar --spring.profiles.active=prod
```

- ##### 在 Java 代码中激活 profile

直接指定环境变量来激活 profile：

```java
System.setProperty("spring.profiles.active", "test");
```

- **在 Spring 容器中激活 profile：**

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

profile还可以用在类上，spring提供了@Peofile注解可以实现不同环境下配置参数的切换，任何@Component或@Configuration注解的类都可以使用@Profile注解。

- **在配置类上使用@Profile注解，如下，该配置只会在prod环境下生效**

```java
@Configuration
@Profile("prod")
public class ProductionConfiguration {
    // ...
}
```

如果在实现类上加上@Profile注解，则可以实现注入接口时根据当时的配置环境动态注入对应的实现类。下面是一个例子：
有一个HelloService接口

```java
public interface HelloService {
    String hello();
}
```

对HelloService接口做了两个实现，分别对应于生产环境和开发环境，如下：

```java
/**
 * 生产环境实现类
 */
@Service
@Profile("dev")
public class DevServiceImpl implements HelloService {

    @Override
    public String hello() {
        return "use dev";
    }
}

/**
 * 开发环境实现类
 */
@Service
@Profile("prod")
public class ProdServiceImpl implements HelloService {

    @Override
    public String hello() {
        return "use prod";
    }
}
```

然后写一个接口调用HelloService：

```java
@RestController
public class HelloController {

    @Autowired
    private HelloService helloService;

    @RequestMapping("hello")
    public String sayHello(){
        return helloService.hello();
    }
}
```

当前启用的配置环境是prod，application.yml配置如下：

```yaml
spring:
  profiles:
    active: prod
```

启动项目，浏览器访问[http://localhost:8082/hello](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8082%2Fhello)，接口返回`use prod`，再改变application.yml配置，启用dev，重启项目，再次访问接口返回`use dev`，说明@Profile注解起到了作用，实现了根据当时的配置环境动态的选择相应的实现类。

### 5.profile解析

在Spring Boot启动的过程中执行其run方法时，会执行如下一段代码：

```java
public class SpringApplication {    
    public ConfigurableApplicationContext run(String... args) {
        ......
        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
            ......
        }
        ......
        return context;
    }
}
```

其中prepareEnvironment方法的相关代码如下：

```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
        ApplicationArguments applicationArguments) {
    ......
    listeners.environmentPrepared(environment);
    ......
    return environment;
}
```

在prepareEnvironment方法处理业务的过程中会调用SpringApplicationRunListeners的environmentPrepared方法发布事件。该方法会遍历注册在spring.factories中SpringApplicationRunListener实现类，然后调用其environmentPrepared方法。

其中spring.factories中SpringApplicationRunListener实现类注册为：

```properties
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```

```java
public void environmentPrepared(ConfigurableEnvironment environment) {
    for (SpringApplicationRunListener listener : this.listeners) {
        listener.environmentPrepared(environment);
    }
}
```

其中listeners便是注册的类的集合，这里默认只有EventPublishingRunListener。它的environmentPrepared方法实现为：

```java
@Override
public void environmentPrepared(ConfigurableEnvironment environment) {
    this.initialMulticaster
            .multicastEvent(new ApplicationEnvironmentPreparedEvent(this.application, this.args, environment));
}
```

其实就是发布了一个监听事件。该事件会被同样注册在spring.factories中的ConfigFileApplicationListener监听到：

```properties
# Application Listeners
org.springframework.context.ApplicationListener=\
......
org.springframework.boot.context.config.ConfigFileApplicationListener,\
......
```

spring boot 3中这个类不在了，似乎是用BootstrapConfigFileApplicationListener。

关于配置文件的核心处理便在ConfigFileApplicationListener中完成。在该类中我们可以看到很多熟悉的常量：

```java
public class ConfigFileApplicationListener implements EnvironmentPostProcessor, SmartApplicationListener, Ordered {

    private static final String DEFAULT_PROPERTIES = "defaultProperties";

    // Note the order is from least to most specific (last one wins)
    private static final String DEFAULT_SEARCH_LOCATIONS = "classpath:/,classpath:/config/,file:./,file:./config/";

    private static final String DEFAULT_NAMES = "application";
}
```

比如Spring Boot默认寻找的配置文件的名称、默认扫描的类路径等。

不仅如此，该类还实现了监听器SmartApplicationListener接口和EnvironmentPostProcessor接口也就是拥有了监听器和环境处理的功能。

其onApplicationEvent方法对接收到事件进行判断，如果是ApplicationEnvironmentPreparedEvent事件则调用onApplicationEnvironmentPreparedEvent方法进行处理，代码如下：

```java
@Override
public void onApplicationEvent(ApplicationEvent event) {
    if (event instanceof ApplicationEnvironmentPreparedEvent) {
        onApplicationEnvironmentPreparedEvent((ApplicationEnvironmentPreparedEvent) event);
    }
    if (event instanceof ApplicationPreparedEvent) {
        onApplicationPreparedEvent(event);
    }
}
```

onApplicationEnvironmentPreparedEvent会获取到对应的EnvironmentPostProcessor并调用其postProcessEnvironment方法进行处理。而loadPostProcessors方法获取的EnvironmentPostProcessor正是在spring.factories中配置的当前类。

```java
private void onApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
    List<EnvironmentPostProcessor> postProcessors = loadPostProcessors();
    postProcessors.add(this);
    AnnotationAwareOrderComparator.sort(postProcessors);
    for (EnvironmentPostProcessor postProcessor : postProcessors) {
        postProcessor.postProcessEnvironment(event.getEnvironment(), event.getSpringApplication());
    }
}
```

经过一系列的调用，最终调用到该类的如下方法：

```java
protected void addPropertySources(ConfigurableEnvironment environment, ResourceLoader resourceLoader) {
    RandomValuePropertySource.addToEnvironment(environment);
    new Loader(environment, resourceLoader).load();
}
```

其中Loader类为ConfigFileApplicationListener内部类，提供了具体处理配置文件优先级、profile、加载解析等功能。

PropertySourceLoader同样配置在spring.factories中：

```yaml
# PropertySource Loaders
org.springframework.boot.env.PropertySourceLoader=\
org.springframework.boot.env.PropertiesPropertySourceLoader,\
org.springframework.boot.env.YamlPropertySourceLoader
```

查看这两PropertySourceLoader的源代码，会发现SpringBoot默认支持的配置文件格式及解析方法。

比如PropertiesPropertySourceLoader中支持了xml和properties两种格式的配置文件，并分别提供了PropertiesLoaderUtils和OriginTrackedPropertiesLoader两个类进行相应的处理。

同样的YamlPropertySourceLoader支持yml和yaml格式的配置文件，并且采用OriginTrackedYamlLoader类进行解析。

当然，在ConfigFileApplicationListener类中还实现了上面提到的如何拼接默认配置文件和profile的实现，相关代码如下：

```java
private void loadForFileExtension(PropertySourceLoader loader, String prefix, String fileExtension,
        Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
    DocumentFilter defaultFilter = filterFactory.getDocumentFilter(null);
    DocumentFilter profileFilter = filterFactory.getDocumentFilter(profile);
    if (profile != null) {
        // Try profile-specific file & profile section in profile file (gh-340)
        String profileSpecificFile = prefix + "-" + profile + fileExtension;
        load(loader, profileSpecificFile, profile, defaultFilter, consumer);
        load(loader, profileSpecificFile, profile, profileFilter, consumer);
        // Try profile specific sections in files we've already processed
        for (Profile processedProfile : this.processedProfiles) {
            if (processedProfile != null) {
                String previouslyLoaded = prefix + "-" + processedProfile + fileExtension;
                load(loader, previouslyLoaded, profile, profileFilter, consumer);
            }
        }
    }
    // Also try the profile-specific section (if any) of the normal file
    load(loader, prefix + fileExtension, profile, profileFilter, consumer);
}
```



## 七、spring boot的异常报告器类

```java
@FunctionalInterface
public interface SpringBootExceptionReporter {

    /**
     * Report a startup failure to the user.
     * @param failure the source failure
     * @return {@code true} if the failure was reported or {@code false} if default
     * reporting should occur.
     */
    boolean reportException(Throwable failure);

}
```

调用情况：

```java
public ConfigurableApplicationContext run(String... args) {
    ......
    //初始化构造一个SpringBootExceptionReporter集合
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    
    try {
        ...
        //给exceptionReporters集合填充具体的对象
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                new Class[] { ConfigurableApplicationContext.class }, context);
        ...
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    ...
    return context;
}
```

上面是在spring boot2中的调用情况，springboot3中如下：

```java
public ConfigurableApplicationContext run(String... args) {
    ......
    //初始化构造一个SpringBootExceptionReporter集合
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    
    try {
        ...
        ...
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        ...
    }

    ...
    return context;
}


private void handleRunFailure(ConfigurableApplicationContext context, Throwable exception, SpringApplicationRunListeners listeners) {
        try {
            try {
                ...
            } finally {
                this.reportFailure(this.getExceptionReporters(context), exception);
                ...

            }
        } catch (Exception var8) {
            ...
        }

        ...
    }

    private Collection<SpringBootExceptionReporter> getExceptionReporters(ConfigurableApplicationContext context) {
        try {
            SpringFactoriesLoader.ArgumentResolver argumentResolver = ArgumentResolver.of(ConfigurableApplicationContext.class, context);
            return this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, argumentResolver);
        } catch (Throwable var3) {
            return Collections.emptyList();
        }
    }
```

通过SpringFactoriesLoader加载:

```
# Error Reporters
org.springframework.boot.SpringBootExceptionReporter=\
org.springframework.boot.diagnostics.FailureAnalyzers
```

SpringBoot通过SPI技术为SpringBootExceptionReporter接口指定了唯一的实现FailureAnalyzers.

FailureAnalyzers实现的SpringBootExceptionReporter的异常报告方法:

```java
@Override
public boolean reportException(Throwable failure) {
    FailureAnalysis analysis = analyze(failure, this.analyzers);
    return report(analysis, this.classLoader);
}

private FailureAnalysis analyze(Throwable failure, List<FailureAnalyzer> analyzers) {
    for (FailureAnalyzer analyzer : analyzers) {
        try {
            // 依次遍历每个analyzer分析,有结果就返回,这边走到AbstractFailureAnalyzer的analyze方法
            FailureAnalysis analysis = analyzer.analyze(failure);
            if (analysis != null) {
                return analysis;
            }
        }
        catch (Throwable ex) {
            if (logger.isDebugEnabled()) {
                logger.debug("FailureAnalyzer " + analyzer + " failed", ex);
            }
        }
    }
    return null;
}
```

AbstractFailureAnalyzer实现了FailureAnalyzer接口

```java
public abstract class AbstractFailureAnalyzer<T extends Throwable> implements FailureAnalyzer {

    @Override
    public FailureAnalysis analyze(Throwable failure) {
        // 会先走到这里
        T cause = findCause(failure, getCauseType());
        if (cause != null) {
            // 找到子类感兴趣的异常后,调用具体实现类的分析方法
            return analyze(failure, cause);
        }
        return null;
    }

    protected abstract FailureAnalysis analyze(Throwable rootFailure, T cause);

    // 这个就是解析我的泛型参数,表明我对哪个异常感兴趣
    protected Class<? extends T> getCauseType() {
        return (Class<? extends T>) ResolvableType.forClass(AbstractFailureAnalyzer.class, getClass()).resolveGeneric();
    }

    protected final <E extends Throwable> E findCause(Throwable failure, Class<E> type) {
        // 依次遍历异常堆栈,如果有对当前异常感兴趣,就返回,否则返回null
        while (failure != null) {
            if (type.isInstance(failure)) {
                return (E) failure;
            }
            failure = failure.getCause();
        }
        return null;
    }
}
```

看看report(analysis, this.classLoader)方法

```java
// 分析出结果之后,调用report方法之后报告异常
private boolean report(FailureAnalysis analysis, ClassLoader classLoader) {
    // FailureAnalysisReporter这个实现类只有一个,是LoggingFailureAnalysisReporter
    List<FailureAnalysisReporter> reporters = SpringFactoriesLoader.loadFactories(FailureAnalysisReporter.class,
            classLoader);
    if (analysis == null || reporters.isEmpty()) {
        return false;
    }
    for (FailureAnalysisReporter reporter : reporters) {
        // 输出异常报告
        reporter.report(analysis);
    }
    return true;
}
```

```java
public final class LoggingFailureAnalysisReporter implements FailureAnalysisReporter {

    private static final Log logger = LogFactory.getLog(LoggingFailureAnalysisReporter.class);

    @Override
    public void report(FailureAnalysis failureAnalysis) {
        if (logger.isDebugEnabled()) {
            logger.debug("Application failed to start due to an exception", failureAnalysis.getCause());
        }
        if (logger.isErrorEnabled()) {
            logger.error(buildMessage(failureAnalysis));
        }
    }

    private String buildMessage(FailureAnalysis failureAnalysis) {
        StringBuilder builder = new StringBuilder();
        builder.append(String.format("%n%n"));
        builder.append(String.format("***************************%n"));
        builder.append(String.format("APPLICATION FAILED TO START%n"));
        builder.append(String.format("***************************%n%n"));
        builder.append(String.format("Description:%n%n"));
        builder.append(String.format("%s%n", failureAnalysis.getDescription()));
        if (StringUtils.hasText(failureAnalysis.getAction())) {
            builder.append(String.format("%nAction:%n%n"));
            builder.append(String.format("%s%n", failureAnalysis.getAction()));
        }
        return builder.toString();
    }

}
```

- ## SpringBoot异常处理流程

```java
public ConfigurableApplicationContext run(String... args) {
    ......
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    ......
    try {
        ......
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                new Class[] { ConfigurableApplicationContext.class }, context);
        ......
    }
    catch (Throwable ex) {
        //对启动过程中的失败进行处理
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    ......
    return context;
}
```

点进去handleRunFailure(context, ex, exceptionReporters, listeners)

```java
private void handleRunFailure(ConfigurableApplicationContext context, Throwable exception,
        Collection<SpringBootExceptionReporter> exceptionReporters, SpringApplicationRunListeners listeners) {
    try {
        try {
            handleExitCode(context, exception);
            if (listeners != null) {
                listeners.failed(context, exception);
            }
        }
        finally {
            reportFailure(exceptionReporters, exception);
            if (context != null) {
                context.close();
            }
        }
    }
    catch (Exception ex) {
        logger.warn("Unable to close ApplicationContext", ex);
    }
    ReflectionUtils.rethrowRuntimeException(exception);
}
```

![image-20240315143128370](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20240315143128370.png)

**handleExitCode(context, exception)**

```
private void handleExitCode(ConfigurableApplicationContext context, Throwable exception) {
    // 获取框架内对这个异常定义的exitCode,代表退出状态码,为0代表正常退出,不为0标识异常退出
    int exitCode = getExitCodeFromException(context, exception);
    if (exitCode != 0) {
        if (context != null) {
            // 发布一个ExitCodeEvent事件
            context.publishEvent(new ExitCodeEvent(context, exitCode));
        }
        SpringBootExceptionHandler handler = getSpringBootExceptionHandler();
        if (handler != null) {
            //记录exitCode
            handler.registerExitCode(exitCode);
        }
    }
}
```

**listeners.failed(context, exception)**

- 发布ApplicationFailedEvent事件

**reportFailure(exceptionReporters, exception)**

- SpringBootExceptionReporter实现类调用reportException方法
- 成功处理的话，记录已处理异常

**context.close()**

- 更改应用上下文状态
- 销毁单例bean
- 将beanFactory置为空
- 关闭web容器(web环境)
- 移除shutdownHook

**ReflectionUtils.rethrowRuntimeException(exception)**

- 重新抛出异常



## 八、spring 的shutdownHook介绍

作用：JVM退出时执行的业务逻辑

添加：Runtime.getRuntime().addShutdownHook()

移除：Runtime.getRuntime().removeShutdownHook(this.shutdownHook)

### 1.背景
 在开发中，遇到这种情况，多个线程同时工作，突然一个线程遇到了fetal的错误，需要立即终止程序，等人工排查解决了问题之后重新启动。但是这样会有一个问题，程序终止时，其他线程可能正在进行重要操作，比如发一个message到另一个模块，并更新数据库状态。突然终止，可能会让这个操作只完成一半，从而导致数据不一致。

解决方案是：参考数据库Transaction原子性的概念，将这一系列重要操作看作一个整体，要么全部完成，要么全部不完成。为方便表述，我们把这一系列重要操作记为操作X。
 当程序即将退出时，查看当前是否有操作X在执行中，

如果有，等待其完成然后退出。且期间不再接受新的操作X。如果操作X执行之间过长，终止并回滚所有状态。
 如果没有，则可以立即退出。
 在程序退出的时候，做一些Check，保证已经开始的操作X的原子性，这里就用到了Runtime.ShutdownHook。

### 2.什么是Shutdown Hook

Shutdown hook是一个initialized but unstarted thread。当JVM开始执行shutdown sequence时，会并发运行所有registered Shutdown Hook。这时，在Shutdown Hook这个线程里定义的操作便会开始执行。

需要注意的是，在Shutdown Hook里执行的操作应当是不太耗时的。因为在用户注销或者操作系统关机导致的JVM shutdown的例子中，系统只会预留有限的时间给未完成的工作，超时之后还是会强制关闭。

### 3.什么时候会调用Shutdown Hook

##### 程序正常停止

- Reach the end of program
- System.exit

##### 程序异常退出

- NPE
- OutOfMemory

##### 受到外界影响停止

- Ctrl+C
- 用户注销或者关机

### 4.如何使用Shutdown Hook

调用java.lang.Runtime这个类的addShutdownHook(Thread hook)方法即可注册一个Shutdown Hook，然后在Thread中定义需要在system exit时进行的操作。如下：

```java
Runtime.getRuntime().addShutdownHook(new Thread(() -> System.out.println("Do something in Shutdown Hook")));
```

### 5.需要注意的点

- 当System.exit之后，当Shutdown Hook开始执行时，其他的线程还是会继续执行。

- 应当保证Shutdown Hook的线程安全。

- 在使用多个Shutdown Hook时一定要特别小心，保证其调用的服务不会被其他Hook影响。否则会出现当前Hook所依赖的服务被另外一个Hook终止了的情况。

## 九、spring boot starter

https://www.jianshu.com/p/ffca6ced7562

## 十、SpringBoot—接口返回结果及异常统一处理

https://www.jianshu.com/p/94dec76d5c73

## 十一、过滤器、监听器、拦截器

https://www.jianshu.com/p/dd1d0e786749

## 十二、异步开发之异步请求

https://www.jianshu.com/p/2ae6904fa936

## 十三、异步开发之异步调用

### Async异步调用

https://www.jianshu.com/p/600ea066352d

**异步@Async失效原因**

https://blog.csdn.net/weter_drop/article/details/135432068?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7EPosition-2-135432068-blog-134643925.235%5Ev43%5Epc_blog_bottom_relevance_base4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7EPosition-2-135432068-blog-134643925.235%5Ev43%5Epc_blog_bottom_relevance_base4&utm_relevant_index=5

## 十四、SpringBoot 事件的发布和监听

https://www.jianshu.com/p/873458c6c45a

## 十五、SpringBoot 启动初始化数据

https://www.jianshu.com/p/01e08aef73c9

### 1.ApplicationRunner 与 CommandLineRunner 接口

### 2.Spring容器初始化时InitializingBean接口和@PostConstruct

### 3.Spring的事件机制

## 十六、定时任务的使用

https://www.jianshu.com/p/758862627867

### 1.Timer

### 2.ScheduledExecutorService

### 3.Spring Task

### 4.Quartz/XXL-JOB

## 十七、Spring Cache

https://blog.csdn.net/weixin_42972832/article/details/127822968



## 十八、Spring Session

https://blog.csdn.net/i_love_t/article/details/127654874

https://blog.csdn.net/qq_49841284/article/details/134045722

## 十九、业务系统忙如何解决

- 优化数据库结构：对于业务系统来说，数据库通常是承担了很大一部分压力，因此优化数据库结构对于提升系统性能至关重要。可以考虑对数据库进行分库分表，将大表拆分成多个小表，避免单表数据量过大而导致的性能瓶颈。

  其次，可以采用缓存技术来减轻数据库压力。通过合理地利用缓存，可以有效降低数据库的访问频率，提升系统的并发处理能力。可以考虑使用内存缓存、分布式缓存等方式来实现缓存。

  最后，需要定期对数据库进行优化和维护，确保数据库的健康运行。可以通过定期清理无用数据、优化SQL语句、设置合理的索引等方式来提升数据库性能。

- 系统架构优化：可以考虑引入分布式架构，将系统拆分成多个独立的子系统，每个子系统可以独立运行，从而提升系统的并发处理能力。

  可以考虑利用消息中间件来实现系统解耦，降低系统之间的耦合度，提升系统的稳定性和性能。通过消息中间件，可以实现异步处理、削峰填谷等功能，从而有效减轻系统压力。

  最后，需从容灾备份的角度进行优化，实现高可用性和容灾能力。可以考虑引入集群技术、数据备份、故障切换等方式来提升系统的稳定性。

- 代码质量不高：可以从代码优化的角度着手进行改进。首先，可以进行代码重构，去除冗余代码，优化算法，提升代码质量。其次，可以引入代码静态分析工具，发现并优化潜在的性能瓶颈。另外，可以加强代码评审，建立良好的编码规范，避免低效的代码写法。

  此外，可以引入性能监控工具，实时监控系统性能指标，发现系统瓶颈。通过性能监控工具，可以快速定位系统性能问题，进行针对性调优。

- 引入高性能服务器，横向扩展的方式，增加服务器节点，提升系统的并发处理能力

- 采用负载均衡技术，将请求按照一定的规则分发到多个服务器上，从而避免单台服务器被过多请求压垮。通过负载均衡，可以实现请求的分流，提升系统的整体吞吐能力。

- 注重系统容量规划。通过对系统容量进行科学规划，可以避免系统因负载过大而瘫痪。可以通过对业务量、并发量等指标进行预估，合理规划系统的容量，保障系统的稳定运行。

## 二十、提高接口性能

![image](../../../../../picbed/store/picbed/img/image-20240425162603223.png)

https://blog.csdn.net/weixin_45433031/article/details/133795171

