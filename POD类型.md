# C++ 11 新特性之POD类型



原文链接：https://www.cnblogs.com/Braveliu/p/12237340.html

https://blog.csdn.net/Jxianxu/article/details/80524526







## 【1】什么是POD类型？

Plain old data structure，缩写为POD，Plain代表是一种普通类型，Old体现该类型的对象可以与C兼容。

POD类型是C++语言标准中定义的一类数据结构，适用于需要明确的数据底层操作的系统中。

POD通常被用在系统的边界处，即指不同系统之间只能以底层数据的形式进行交互，系统的高层逻辑不能互相兼容。

比如，当对象的字段值是从外部数据中构建时，系统还没有办法对对象进行语义检查和解释，这时就适用POD来存储数据。

严格来讲，一种既是普通类型（Trivial Type）又是标准布局类型（Standard-layout Type）的类型，即是POD类型。

通俗地讲，一种类型的对象通过二进制拷贝（如memcpy()）后还能保持数据不变可正常使用的类型，即是POD类型。



## 【2】为什么引入POD类型？

不同类型的对象，本质区别在于对象的成员在内存中的布局是不同的。

在某些情况下，布局是有明确规范的定义，但如果类或结构包含某些C++语言功能，如虚基类、虚函数、具有不同的访问控制的成员等，

则不同编译器会有不同的布局实现，具体取决于编译器对代码的优化方式，比如实现内存对齐，减少访存指令周期等。

例如，如果类具有虚函数，该类的所有实例都会包含一个指向虚函数表的指针，那么这个对象就不能直接通过二进制拷贝的方式传到其它语言编程的程序中使用。

C++给定对象的类型取决于其特定的内存布局方式，一个对象是普通、标准布局还是POD类型，可以根据标准库函数模板来判断：

```c++
#include <type_traits>
#include <iostream>
using namespace std;

int main()
{
    cout << is_trivial<int>::value << endl;          // 1
    cout << is_standard_layout<int>::value << endl;  // 1
    cout << is_pod<int>::value << endl;              // 1

    system("pause");
}
```

POD对象与C语言中的对象具有一些共同的特性，包括初始化、复制、内存布局与寻址：

（1）可以使用字节赋值，比如用memset、memcpy对POD类型进行赋值操作；

（2）对C内存布局兼容，POD类型的数据可以使用C函数进行操作且总是安全的；

（3）保证了静态初始化的安全有效，静态初始化可以提高性能，如将POD类型对象放入BSS段默认初始化为0。



## 【3】POD类型需要满足什么条件？

**POD类型既要满足平凡类型的条件又要满足标准布局类型的条件。**

### 平凡性（trivial）

什么是平凡性呢？通常一个平凡的类或者结构体具有以4点下特征：

1.      具有平凡的默认构造函数。如果我们不自己为类定义任何构造函数，编译器就会为我们产生一个平凡的默认构造函数；一旦我们为类定义了任何一种构造函数，那这个构造函数就不是平凡的。

2.      具有平凡的拷贝构造函数和移动构造函数。即拥有编译器自动生成的拷贝、移动构造函数。

3.      拥有平凡的拷贝赋值运算符和移动赋值运算符。

4.      不能包含虚函数和虚基类。

C++11提供了一个类模板来帮我们识别一个类是否是平凡的：

```c++
#include <type_traits>
#include <iostream>
using namespace std;

class A { A() {} };
class B { B(B&) {} };
class C { C(C&&) {} };
class D { D operator=(D&) {} };
class E { E operator=(E&&) {} };
class F { ~F() {} };
class G { virtual void foo() = 0; };
class H : G {};
class I {};

int main()
{
    std::cout << std::is_trivial<A>::value << std::endl;  // 有不平凡的构造函数
    std::cout << std::is_trivial<B>::value << std::endl;  // 有不平凡的拷贝构造函数
    std::cout << std::is_trivial<C>::value << std::endl;  // 有不平凡的拷贝赋值运算符
    std::cout << std::is_trivial<D>::value << std::endl;  // 有不平凡的拷贝赋值运算符
    std::cout << std::is_trivial<E>::value << std::endl;  // 有不平凡的移动赋值运算符
    std::cout << std::is_trivial<F>::value << std::endl;  // 有不平凡的析构函数
    std::cout << std::is_trivial<G>::value << std::endl;  // 有虚函数
    std::cout << std::is_trivial<H>::value << std::endl;  // 有虚基类

    std::cout << std::is_trivial<I>::value << std::endl;  // 平凡的类

    system("pause");
    return 0;
}

/*运行结果
0
0
0
0
0
0
0
0
1
*/
```



### 标准布局

满足以下条件的类或结构体是标准布局的

1.所有非静态成员有相同的访问权限，比如都是private的，或者都是public，或者都是protected

2. 在类或结构体的继承时，满足以下两种情况之一：

   a.      派生类中有非静态成员，且只有仅包含静态成员的基类。

   b.      基类有非静态成员，而派生类没有非静态成员。

   这两条综合起来，意即：继承树中最多只能有一个类有非静态数据成员。

3. 类中第一个非静态成员的类型与其基类不同。这条规则是基于C++中优化不包含成员的基类而产生的。请看下面的例子：

   ```c++
   class B1{};
   class B2{};
    
   class D1: public B1
   {
       B1 b;
       int i ;
   };
   class D2: public B1
   {
       B2 b ;
       int i ;
   };
   ```

   B1和B2两个基类中不包含任何数据成员，B1的子类D1中的第一个非静态成员的类型和其基类相同，B1的子类D2中第一个非静态成员变量的类型是B2，与其基类并不相同。由于B1和B2都不包含任何数据成员，这样看起来D1和D2两个类的类对象占用的内存空间应该是一样的。但实际则不然，在C++标准中，如果基类没有任何数据成员，基类应不占用空间，为了体现这一点，C++标准允许派生类的第一个成员与基类共享同一地址空间。但是如果派生类的第一个非静态成员的类型和基类相同，由于C++标准要求相同类型的对象的地址必须不相同，编译器就会为基类分派一个字节的地址空间。所以在此例中，D1和D2的内存布局其实是不相同的，请看下图：

   ![img](E:\Debug_cpp\POD类型.assets\2018053115095940.png)

4. 没有虚函数和虚基类。

5. 所有非静态成员都符合标准布局类型，其父类也符合标准布局。

```c++
#include <type_traits>
#include <iostream>
using namespace std;

class A
{
private:
    int a;
public:
    int b;
};

class B1
{
    static int x1;
};

class B2
{
    int x2;
};

class B : B1, B2
{
    int x;
};

class C1 {};
class C : C1
{
    C1 c;
};

class D { virtual void foo() = 0; };
class E : D {};
class F { A x; };

int main()
{
    std::cout << std::is_standard_layout<A>::value << std::endl;  // 违反定义1：成员a和b具有不同的访问权限
    std::cout << std::is_standard_layout<B>::value << std::endl;  // 违反定义2：继承树有两个(含)以上的类有非静态成员
    std::cout << std::is_standard_layout<C>::value << std::endl;  // 违反定义3：第一个非静态成员是基类类型
    std::cout << std::is_standard_layout<D>::value << std::endl;  // 违反定义4：有虚函数
    std::cout << std::is_standard_layout<E>::value << std::endl;  // 违反定义5：有虚基类
    std::cout << std::is_standard_layout<F>::value << std::endl;  // 违反定义6：非静态成员x不符合标准布局类型

    system("pause");
    return 0;
}

/*运行结果
0
0
0
0
0
0
*/
```



C++11提供了如下模板来判断一个类或结构体对象是否是标准布局

```c++
template <typename T> structstd::is_standard_layout; //头文件为<type_traits>
template <typename T> struct std::is_pod //判断一个类型是否是POD，头文件为<type_traits>
```

POD的好处：

1 字节赋值，我们可以放心的使用memset和memcpy对POD类型进行初始化和拷贝。

2 提供对C内存的兼容。POD类型的数据在C与C++间的操作总是安全的。

3 保证了静态初始化的安全有效。POD类型的对象初始化往往更简单



## 【4】POD类型的应用

当一个数据类型满足了“平凡的定义”和“标准布局定义”，我们则认为它是一个POD数据。

一个POD类型是可以进行二进制拷贝的，如下代码示例：

```c++
#include <type_traits>
#include <iostream>
using namespace std;

class A
{
public:
    int x;
    double y;
};

int main()
{
    if (std::is_pod<A>::value)
    {
        std::cout << "before" << std::endl;
        A a;
        a.x = 8;
        a.y = 10.5;
        std::cout << a.x << std::endl;
        std::cout << a.y << std::endl;

        size_t size = sizeof(a);
        char* p = new char[size];
        memcpy(p, &a, size);
        A* pA = (A*)p;

        std::cout << "after" << std::endl;
        std::cout << pA->x << std::endl;
        std::cout << pA->y << std::endl;

        delete p;
    }

    system("pause");
    return 0;
}

/*运行结果
before
8
10.5
after
8
10.5
*/
```

很明显，对一个POD类型的对象进行二进制拷贝后，数据可成功迁移。