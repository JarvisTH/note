# using关键字用法总结

## 一、命名空间

配合命名空间，对命名空间权限进行管理。using + 命名空间名字，这样可以在程序中直接用命令空间中的类型，而不必指定类型的详细命名空间，类似于Java的import，这个功能也是最常用的。

```c++
using namespace std;//释放整个命名空间到当前作用域
using std::cout;    //释放某个变量到当前作用域
```

## 二、指定别名

`using`的另一个作用是指定别名，一般都是`using a = b;`这样的形式出现：

```c++
using ModuleType = ClassOne;
```

`ModuleType` 是`ClassOne`的一个别名。

## 三、在子类中引用基类的成员

`using`的第三个作用是子类中引用基类的成员，一般都是`using CBase::a;`这样的形式出现。

```c++
using typename ClassType::ModuleType;
```

加了个`typename` 修饰，这是因为类`ClassThree`本身是个模板类，它的基类`ClassType`是个模板，这个`typename` 和`using`其实没有什么关系。如果`ClassType`不是模板的话，这行代码就可以写成：

```c++
using ClassType::ModuleType;
```

剩下的就是using的作用，它引用了基类中的成员`ModuleType`， `ModuleType`又是类`ClassOne`的别名，所以后面`ModuleType m;`相当于定义对象m，对于子类成员m来说，这样的效果和下面是相同的：

```c++
typename ClassType::ModuleType m;
```

不同之处在于`using`还修改了基类成员的访问权限，子类`ClassThree` 私有继承`ClassType`，所以`ClassType`中共有成员`ModuleType` 在子类`ClassThree` 是私有的，它不能被外部访问。但是使用`using`后，在`main()`函数中可以使用。

参考：https://www.cnblogs.com/2018shawn/p/13565172.html