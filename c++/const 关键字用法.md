# const关键字总结

**const** 是**constant**的缩写，本意是**不变的，不易改变**的意思。

**const** 在C++中是用来修饰**内置类型变量，自定义对象，成员函数，返回值，函数参数**。

## 一、const修饰普通类型变量

```c++
const int a = 7;
int b= a;
a = 8;
```

a被定义为一个常量，并且可以将a赋值给b，但是不能给a再次赋值。对一个常量赋值是违法的事情，因为**a被编译器认为是一个常量**，其值**不允许修改**。

**Volatile**关键字跟**const对应相反，是易变的，容易改变**的意思。在const前面加上**volatile**关键字,不让编译器察觉到对const的操作，不会被编译器优化，编译器也就不会改变对a变量的操作。不加volatile关键字，下面的输出是7。

```c++
#include<iostream>

using namespace std;

int main(void)
{

    volatile const int  a = 7;
    int  *p = (int*)&a;

    *p = 8;

    cout<<a;

    system("pause");
    return 0;
}
```

## 二、const 修饰指针变量

const 修饰指针变量有以下三种情况：

- **修饰指针指向的内容，则内容为不可变量**
- **修饰指针，则指针为不可变量**
- **修饰指针和指针指向的内容，则指针和指针指向的内容都为不可变量**

对于**修饰指针指向的内容**，如下：

```c++
const int *p=8;
```

则指针指向的内容8不可改变。简称**左定值**，因为**const位于\*号的左边**。



对于**修饰指针**，如下：

```c++
int a = 8;
int* const p = &a;

*p = 9; //it’s right

int  b = 7;
p = &b; //it’s wrong
```

对于const指针p其指向的内存地址不能够被改变，但其内容可以改变。简称，**右定向**。因为**const位于 \* 号的右边**。

对于**饰指针和指针指向的内容**，如下：

```c++
int a = 8;
const int * const  p = &a;
```

const p的指向的内容和指向的内存地址都已固定，不可改变。

根据const位于*号的位置不同，**左定值，右定向，const修饰不变量**。

## 三、const int function参数传递

对于const修饰函数参数可以分为三种情况：

- ### 值传递的const修饰传递

一般这种情况不需要const修饰，因为**函数会自动产生临时变量复制实参值**。

```c++
#include<iostream>
using namespace std;

void Cpf(const int a)
{
    cout<<a; 
    // ++a;  it's wrong, a can't is changed
}

int main(void)
{
    Cpf(8);

    system("pause"); 
    return 0;
}
```

- ### const参数为指针

**当const参数为指针时，可以防止指针被意外篡改**。

```c++
#include<iostream>
using namespace std;

void Cpf(int *const a) 
{ 
    cout<<*a<<" ";
    *a = 9;
}

int main(void)
{ 
    int a = 8; 
    Cpf(&a);

    cout<<a; // a is 9 
    system("pause");
    return 0;
}
```

- ### self definded type, const reference

**自定义类型的参数传递，需要临时对象复制参数，对于临时对象的构造，需要调用构造函数，比较浪费时间，因此我们采取const外加引用传递的方法。**

并且对于一般的int ，double等内置类型，我们不采用引用的传递方式。

```c++
#include<iostream>

using namespace std;

class Test 
{ 
public: 
    Test(){} 

    Test(int _m):_cm(_m){}

    int get_cm() const 
    {
       return _cm;
    }

private:
    int _cm;
};


void Cmf(const Test& _tt)
{
    cout<<_tt.get_cm(); 
}

int main(void) 
{
    Test t(8);

    Cmf(t);

    system("pause");
    return 0;
}
```

## 四、const修饰函数的返回值

Const修饰返回值分三种情况：

- **const修饰内置类型的返回值，修饰与不修饰返回值作用一样**

```c++
#include<iostream>

using namespace std;

const int Cmf()
{ 
    return 1; 
}

int Cpf() 
{ 
    return 0;
}

int main(void) 
{
    int _m = Cmf(); 
    int _n = Cpf();

    cout<<_m<<" "<<_n;
    system("pause"); 
    return 0;
}
```

- **const 修饰自定义类型的作为返回值，此时返回的值不能作为左值使用，既不能被赋值，也不能被修改。**
- **const 修饰返回的指针或者引用，是否返回一个指向const的指针，取决于我们想让用户干什么。**

## 五、const修饰类成员函数

const 修饰类成员函数，其目的是防止成员函数修改被调用对象的值。如果我们不想修改一个**调用对象的值**，所有的成员函数都应当声明为const成员函数。

**const关键字不能与static关键字同时使用，因为static关键字修饰静态成员函数，静态成员函数不含有this指针，即不能实例化，const成员函数必须具体到某一实例。**

```c++
#include<iostream>

using namespace std;

class Test
{
public:
    Test(){}
    Test(int _m):_cm(_m){}

    int get_cm()const
    {
       return _cm;
    }

private:
    int _cm;

};

void Cmf(const Test& _tt)
{
    cout<<_tt.get_cm();
}

int main(void)
{
    Test t(8);
    Cmf(t);

    system("pause");
    return 0;
}
```

**如果get_cm()去掉const修饰，则Cmf传递的const _tt即使没有改变对象的值，编译器也认为函数会改变对象的值，所以我们尽量按照要求将所有的不需要改变对象内容的函数都作为const成员函数。**

**如果有个成员函数想修改对象中的某一个成员怎么办？**这时我们可以使用**mutable**关键字修饰这个成员，mutable的意思也是**易变的，容易改变的**意思，被mutable关键字修饰的成员可以处于不断变化中。

```c++
#include<iostream>
using namespace std;

class Test
{
public:
    Test(int _m,int _t):_cm(_m),_ct(_t){}
    void Kf() const
    {
        ++_cm; //it's wrong
        ++_ct; //it's right
    }
private:
    int _cm;
    mutable int _ct;
};

int main(void)
{
    Test t(8,7);
    return 0;
}
```

修改_ct的值，但是通过++_cm修改_cm则会报错。因为++_cm没有用**mutable修饰**。

参考：https://www.cnblogs.com/wshisuifeng/p/10873344.html