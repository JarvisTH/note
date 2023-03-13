**一、Web项目的目录架构**

基于Java的Web应用，标注打包方式是WAR。典型的WAR文件目录结构：
![](https://upload-images.jianshu.io/upload_images/9449419-2b04c35593aec288.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
WAR包至少包含两个子目录：META-INF和WEB-INF。前者包含打包元数据。后者是WAR包核心，必须包含一个Web资源表述文件web.xml，它的子目录classes包含所有该Web项目的类，另一个子目录lib则包含所有该Web项目的依赖JAR包，classes和lib目录都会在运行时被加入到Classpath中。

Maven必须为Web项目显式指定打包方式为war。

Web项目的类及资源文件同Jar项目一样，默认src/main/java/和src/main/resouces/，测试类及测试资源文件的默认src/test/java/和src/test/resources/。Web项目还有一个Web资源目录，默认src/main/webapp/：
![](https://upload-images.jianshu.io/upload_images/9449419-e7d70484087c3912.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在src/main/webapp/目录下，必须包含一个子目录WEB-INF，该子目录必须包含web.xml文件。Maven在用WAR打包时会根据POM的配置从本地仓库复制相应JAR文件。

**二、account-web

**1.account-web的pom**
![](https://upload-images.jianshu.io/upload_images/9449419-643615ca9339669f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/9449419-dbcb7a6d2391350d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
account-web依赖于servlet-api和jsp-api这两个几乎所有Web项目都要依赖的包，为servlet和jsp编写提供支持。依赖范围是provided，表述不会被打包至war文件
，因为几乎所有的Web容器都会提供这两个类库，如果war包出现重复，会导致潜在的依赖冲突。

还依赖account-service和spring-web，前者提供底层支持，后者为Web应用提供Spring集成支持。

需要名字更简洁的war包，配置finalName元素：
```
<finalName>name</finalName>
```

Servlet的配置从web.xml获得，位于src/main/webapp/WEB-INF/目录。

**三、使用jetty-maven-plugin进行测试**

Web页面测试应该仅限于页面层次，如jsp、css等修改，其他代码修改应该进行单元测试。

jetty-maven-plugin能够周期性的检查项目内容，发现变更后自动更新到内置的Jetty Web容器。
```
<plugin>
    <groupId>org.mortbay.jetty</groupId>
    <artifactId>jetty-maven-plugin>
    <version>7.1.6.v20100715</version>
    <configuration>
        <scanIntervalSeconds>10</scanIntervalSeconds>
        <webAppConfig>
            <contextPath>/test</contextPath>
        </webAppConfig>
    </configuration>
</plugin>
```
scanIntervalSeconds表示该插件扫描项目变更的时间间隔。如果不进行配置，默认是0，表示不扫描，失去了自动化热部署功能。contextPath表示项目部署后的context path。

启用jetty-maven-plugin前，需要对settings.xml修改,因为默认仓库下才支持简化命令行调用，mvn jetty:run不行，因为其groupId是org.mortbay.jetty，为了能直接运行mvn jetty:run，需要配置：
```
<settings>
     <pluginGroups>
        <pluginGroup>org.mortbay.jetty</pluginGroup>
    </pluginGroups>
    ...
</settings>
```
启用jetty，默认监听本地8080端口。如果想使用其他端口，添加jetty.port参数：mvn jetty:run -Djetty.port=9999。

**四、使用Cargo实现自动化部署**

Cargo是一组帮助用户操作Web容器的工具，实现自动化部署，几乎支持所有Web容器。Cargo通过cargo-maven2-plugin通过Maven集成。

**1.部署至本地Web容器**

支持两种本地部署方式，分别是standalone模式和existing模式。

在standalone模式下，Cargo会从Web容器的安装目录复制一份配置到用户指定目录，在此基础上部署应用，每次重新构建时，这个目录会被清空，所有配置重新生成。

在existing模式下，用户指定现有的Web容器配置目录，Cargo会直接使用这些配置并将应用部署到其对应的位置。
```
<plugin>
    <groupId>org.codehaus.cargo</groupId>
    <artifactId>cargo-maven2-plugin</artifactId>
    <version>1.0</version>
    <configuration>
        <container>
            <containerId>tomcat6x</contaionerId>
            <home>D:\cmd\apache-tomcat-6.0.29</home>
        </container>
    </configuration>
    <configuration>
        <type>standalone</type>
        <home>${project.build.directory}/tomcat6x</home>
        <properties>
            <cargo.servlet.port>8081</cargo.servlet.port>
        </properties>
    </configuration>
 </plugin>
```
因为其groupId不属于官方groupId，需要添加到settings.xml的pluginGroup元素以方便命令调用：mvn cargo:start。默认端口8080，可配置修改。

**2.部署至远程Web容器**
```
<plugin>
    <groupId>org.codehaus.cargo</groupId>
    <artifactId>cargo-maven2-plugin</artifactId>
    <version>1.0</version>
    <configuration>
        <container>
            <containerId>tomcat6x</contaionerId>
            <type>remote</type>
        </container>
    </configuration>
    <configuration>
        <type>runtime</type>
        <properties>
            <cargo.remote.username>admin</cargo.remote.username>
            <cargo.remote.password>xxx</cargo.remote.username>
            <cargo.remote.url>zzzz</cargo.remote.url>
        </properties>
    </configuration>
 </plugin>
```
远程部署，container元素的type元素必须是remote。如果不显示指定，Cargo会使用默认值installed，并寻找对应的安装目录或安装包，对于远程部署方式，安装目录或安装包是不需要的。

configuration的type元素为runtime，表示既不使用独立的容器配置，也不使用本地现有的容器配置，而是依赖于已运行的容器。

部署命令：mvn cargo:redeploy。

若容器中已经部署了当前应用，Cargo会将其卸载后再重新部署。
