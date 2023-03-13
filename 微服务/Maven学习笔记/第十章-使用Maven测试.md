自动化单元测试，通过maven-surefire-plugin于主流单元测试框架junit3，junit4及TestNG集成。

**一、maven-surefire-plugin简介**

Maven不是一个单元测试框架，主流测试框架是junit，TestNG。Maven所作的是在构建执行到特定生命周期阶段，通过maven-surefie-plugin来执行JUnit或TestNG的测试用例。maven-surefie-plugin，可以称之为测试运行器。

默认情况下，maven-surefire-plugin的test目标会自动执行测试源码路径下所有符合一组命令模式的测试类，模式为：
- **/Test*.java:任何子目录下所有以Test开头的Java类
- **/*Test.java
- **/*TestCase.java
将测试类按上述模式命名，Maven就能自动运行他们。

**三、跳过测试**

在命令行加入参数skipTests可以跳过测试：
mvn package-DskipTests。

也可以在maven-surefire-plugin配置，不推荐。
```
...
<configuration>
    <skipTests>true</skipTests>
 </configuration>
...
```

还运行临时性的跳过测试代码的编译：mvn package-Dmaven.test.skip=true。不推荐。也可以在插件中配置skip为true。

**四、动态指定运行的测试用例**

maven-surefire-plugin提供一个test参数让Maven用户在命令行指定要运行的测试用例。例如，只想运行RandomGeneratorTest，使用：mvn test-Dtest=RandomGeneratorTest。test参数支持符号匹配，使用逗号指定多个测试用例。

test参数必须匹配一个或多个测试类，如果插件找不到匹配的测试类，会报错并构建失败。-DfailIfNoTests=false，告诉插件即使没有任何测试也不报错。

**五、包含与排除测试用例**

使用includes、include包含测试类。
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.5</version>
    <configuration>
        <includes>
            <include>**/*Tests.java</include>
        </includes>
    </configuration>
</plugin>
```
使用excludes、exclude排除测试类。
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.5</version>
    <configuration>
        <excludes>
            <exclude>**/*Tests.java</exclude>
        </excludes>
    </configuration>
</plugin>
```

**六、测试报告**

**1.基本测试报告**

默认，maven-surefire-plugin会在项目的target/surefire-reports目录下生成两种格式的错误报告：
- 简单文本格式
- 与JUnit兼容的XML格式

**2.测试覆盖率报告**

Cobertura——测试覆盖率统计工具，Maven通过cobertura-maven-plugin与之集成，可以使用简单命令生成测试覆盖率报告：mvn cobertura：cobertura。

**七、运行TestNG测试**

在POM中加入TestNG依赖。

![JUnit与TestNG常用类库对应关系](https://upload-images.jianshu.io/upload_images/9449419-604849c8d2779c4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
TestNG运行用户使用testng.xml文件配置想运行的测试集合。
```
<?xml version="1.0" encoding="UTF-8"?>
 <suite name="Suite1" verbose="1">
        <test name="Regression1">
            <classes>
                <class name="xx.xx.xx.xxx"/>
           </classes>
        </test。
  </suite>
```
同时配置maven-surefire-plugin使用该testng.xml：
```
<plugin>
    ...
     <configuration>
        <suiteXmlFiles>
            <suiteXmlFile>testng.xml</suiteXmlFile>
        </suiteXmlFiles>
    </configuration>
</plugin>
```
TestNG支持测试组概念：@Test=（group=（“unit”，“medium”））。可以在插件中配置多个测试组。
```
<plugin>
     ...
     <configuration>
        <groups>unit,medium</groups>
    </configuration>
</plugin>
```

**八、重用测试代码**

配置maven-jar-plugin将测试类打包：
```
<plugin>
    <groupId>org.apache.maven.plugin</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.2</version>
    <executions>
        <execution>
            <goal>test-jar</goal>
        </execution>
    </executions>
 </plugin>
 ```
maven-jar-plugin有两个目标，分别是jar、test-jar，前者内置绑定在default生命周期的package阶段，其行为是对项目主代码打包，后者没有内置绑定，需要显式声明该目标来打包测试代码。test-jar默认绑定生命周期阶段为package。

通过依赖声明使用测试包构建。
```
<dependency>
    <groupId>xxxx</groupId>
    <artifactId>xxxx</artifactId>
    <version>1.0.0-SNAPSHOT</type>
    <type>test-jar</type>
    <scope>test</scope>
</dependency>
```
