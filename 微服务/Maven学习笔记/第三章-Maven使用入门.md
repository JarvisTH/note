**一、编写POM**
POM——Project Object Model，项目对象模型，定义了项目的基本信息，用于描述项目如何构建，声明项目依赖等。

首先编写hello world的pom.xml。
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.jarvis</groupId>
    <artifactId>helloworld</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>Maven Hello World Project</name>

</project>
```
第一行是XML头，指定该xml文档的版本和编码方式。

project元素是所有pom.xml的根元素，还申明了一些POM相关的命名空间及xsd元素。

modeVersion指定了当前POM模型版本，对于Maven2/3来说，只能是4.0.0。

最重要是包含groupId，artifactId，version三行。这三个元素定义了一个项目的基本坐标。在Maven里，任何jar，pom，war都是以这些基本坐标区分的。

groupId定义项目属于哪个组，这个组往往和项目所在的组织或者公司存在关联。

artifactId定义当前Maven项目在组中唯一的Id。

version指定当前项目的版本——1.0-SNAPSHOT。

name元素声明了一个对于用户更友好的项目名称，不是必须的。

Maven能让项目对象模型最大程度的与实际代码独立，即解耦或正交性，很大程度避免了Java代码与pom代码的相互影响。


**二、编写主代码**
项目的主代码会被打包到最终的构件中（如jar），而测试代码只在运行测试时用到，不会被打包。
默认情况下，**Maven假设项目主代码位于src/main/java目录**，Maven会自动搜寻该目录找到项目主代码。
```
package com.jarvis.helloworld;

public class HelloWorld {
    public String sayHello(){
        return "Hello World";
    }

    public static void main(String[] args){
        System.out.println(new HelloWorld().sayHello());
    }
}
```
一般来说，项目中Java类的包都应该基于项目的groupId和artifactId，清晰且符合逻辑，方便搜索。

在IDEA的Maven中运行clean、compile，在控制台会输出如下信息：
```
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Hello World Project 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ helloworld ---
[INFO] Deleting D:\Maven\code\helloworld\target
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.401 s
[INFO] Finished at: 2019-04-29T23:36:30+08:00
[INFO] Final Memory: 10M/35M
[INFO] ------------------------------------------------------------------------
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Hello World Project 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ helloworld ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ helloworld ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to D:\Maven\code\helloworld\target\classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.676 s
[INFO] Finished at: 2019-04-29T23:37:16+08:00
[INFO] Final Memory: 11M/37M
[INFO] ------------------------------------------------------------------------
```
clean告诉Maven清理输出目录target/，compile告诉maven编译项目主代码。
clean任务，删除target/目录。默认情况下，Maven构建的所有输出都在target目录下；接着执行resources；最后执行compile：compile任务，将项目主代码编译至target/classes目录。

**三、编写测试代码**
主代码与测试代码分别位于独立目录中。Maven的默认测试代码目录是src/test/java。
使用JUnit进行单元测试。首先添加JUnit依赖，修改项目的POM代码如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.jarvis</groupId>
    <artifactId>helloworld</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>Maven Hello World Project</name>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.7</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.9</maven.compiler.source>                  
        <maven.compiler.target>1.9</maven.compiler.target>
    </properties>

</project>
```
dependencies元素下可以包含多个dependency元素以声明项目的依赖。Maven自动访问中央仓库，下载需要的文件，也可自己访问该仓库。（加了一段properties元素是因为compile时报错“不再支持源选项 1.5。请使用 1.6 或更高版本。）

scope元素为依赖范围，若依赖范围为test则标识该依赖只对测试有效。如果不声明依赖范围，默认值是compile，表示该依赖对主代码和测试代码都有效。

测试类：测试sayHello方法，检查其返回值是否为“Hello World”。
```
package com.jarvis.helloworld;

import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class HelloWorldTest {
    @Test
    public void testSayHello(){
        HelloWorld helloWorld=new HelloWorld();
        String result=helloWorld.sayHello();
        assertEquals("Hello World",result);
    }
}
```
包含三个步骤：
1.准备测试类及数据；
2.执行要测试的行为；
3.检查结果。
在JUnit 3中约定使用需要执行测试的方法都已test开头。在JUnit 4中需要执行测试的方法都应该以@Test进行标注。

进行maven调试，运行 clean test：
 ```
[INFO] Scanning for projects...                                                                  
.......
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.jarvis.helloworld.HelloWorldTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.06 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 13.254 s
[INFO] Finished at: 2019-04-30T00:00:28+08:00
[INFO] Final Memory: 13M/46M
[INFO] ------------------------------------------------------------------------
```
maven执行了clean，resources：resources，compile：compile，resources：testResources和compiler：testCompile。在Maven执行test之前，会显自动执行项目主资源处理，主代码编译，测试资源处理，测试代码编译等工作，这是maven生命周期特性。显然通过了测试。

注意，书上的结果是编译失败，书上的pom代码没有加那一段properties，给出的原因是compiler插件默认支持编译Java 1.3，所以需要配置该插件使其支持更高版本。添加代码如下：
```
....
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.9</source>
                    <target>1.9</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
...
```

**四、打包和运行**
下一步就是打包，POM中没有指定打包类型，使用默认打包类型jar。执行clean package进行打包，输出如下：
```
...
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ helloworld ---
[INFO] Building jar: D:\Maven\code\helloworld\target\helloworld-1.0-SNAPSHOT.jar
...
```
jar:jar任务负责打包，实际上就是jar插件的jar目标将项目主代码打包成一个名为helloworld-1.0-SNAPSHOT.jar的文件。该文件在target输出目录中，根据artifact-version.jar规则进行命名。可以使用finalName来自定义该文件名称。

**如何让其他maven项目直接引用这个jar呢？**
执行install命令：
```
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ helloworld ---
[INFO] 
[INFO] --- maven-install-plugin:2.4:install (default-install) @ helloworld ---
[INFO] Installing D:\Maven\code\helloworld\target\helloworld-1.0-SNAPSHOT.jar to C:\Users\Jarvis\.m2\repository\com\jarvis\helloworld\1.0-SNAPSHOT\helloworld-1.0-SNAPSHOT.jar
[INFO] Installing D:\Maven\code\helloworld\pom.xml to C:\Users\Jarvis\.m2\repository\com\jarvis\helloworld\1.0-SNAPSHOT\helloworld-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
...
```
package命令将任务项目输出的jar安装到Maven本地仓库中。默认打包生成的jar是不能直接运行的，因为带有main方法的类信息不会添加到manifest中。要生成可执行的jar，需要maven-shade-plugin，配置如下：
```
...
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>1.2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>com.jarvis.helloworld.HelloWorld</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
...
```
配置mainClass为com.jarvis.helloworld.HelloWorld，项目打包时会将该信息放到MANIFEST中。执行clean，install。完成后，可以在target目录看到helloworld-1.0-SNAPSHOT.jar和original-helloworld-1.0-SNAPSHOT.jar，前者带有Main-Class信息的可运行jar，后者是原始jar。

执行helloworld-1.0-SNAPSHOT.jar文件，得到输出。

**五、使用Archetype生成项目骨架**
以上项目的目录结构和pom.xml文件内容称为项目的骨架。
Maven通过Archetype快速勾勒项目骨架。
如果是Maven 3 ，运行：
```
mvn archetype:generate
```
Maven 2，运行：
```
mvn org.apache.maven.plugins:maven-archetype-plugin:2.0-alpha-s:generate
```
实际上是在运行插件maven-archetype-plugin，格式是groupId：artifactId：version：goal。
运行后，得到很多输出。每个Archetype前面都有一个编号，同时命令行会提示一个默认编号，其对应的Archetype为maven-archetype-quickstart，回车选择，然后输入groupId，artifactId，version以及包名package。Archetype插件会根据提供的信息创建项目骨架。
