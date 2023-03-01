https://blog.csdn.net/qq_44732146/article/details/119968376

MapStruct是一个Java注释处理器，用于生成类型安全的bean映射类。

定义一个映射器接口，该接口声明任何必需的映射方法。在编译期间，MapStruct将生成此接口的实现。此实现使用简单的Java方法调用在源对象和目标对象之间进行映射，即没有反射或类似内容。与手动编写映射代码相比，MapStruct通过生成繁琐且易于出错的代码来节省时间。遵循配置方法上的约定，MapStruct使用合理的默认值，但在配置或实现特殊行为时不加理会。

与动态映射框架相比，MapStruct具有以下优点：

1. 通过使用普通方法调用（settter/getter）而不是反射来快速执行
2. 编译时类型安全性：只能映射相互映射的对象和属性，不能将order实体意外映射到customer DTO等。
3. 如果有如下问题，编译时会抛出异常
   3.1 映射不完整（并非所有目标属性都被映射）
   3.2 映射不正确（找不到正确的映射方法或类型转换）
4. 可以通过freemarker定制化开发

## 一、配置

```pom
<!--mapStruct依赖 高性能对象映射-->
            <!--mapstruct核心-->
            <dependency>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct</artifactId>
                <version>1.5.0.Beta1</version>
            </dependency>
            <!--mapstruct编译-->
            <dependency>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.5.0.Beta1</version>
            </dependency>
```

**Lombok依赖：（版本最好在1.16.16以上，否则会出现问题）通常是和lombok一起使用**

```pom
<dependency>
            <groupid>org.projectlombok</groupid>
            <artifactid>lombok</artifactid>
            <version>${lombok.version}</version>
          	// 版本号 1.18.12
        </dependency>
```

下载插件(不是必须的，但是挺好用)
**idea中下载 mapstruct support 插件，安装重启Idea：**

## 二、定义一个映射器

要创建映射器，只需使用所需的映射方法定义一个Java接口，并用注释对其进行org.mapstruct.Mapper注释：
该@Mapper注释将使得MapStruct代码生成器创建的执行PersonMapper 过程中生成时的界面。

在生成的方法实现中，源类型（例如Person）的所有可读属性都将被复制到目标类型（例如PersonDTO）的相应属性中：

1. 当一个属性与其目标实体对应的名称相同时，它将被隐式映射。
2. 当属性在目标实体中具有不同的名称时，可以通过@Mapping注释指定其名称。