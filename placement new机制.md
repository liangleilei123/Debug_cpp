# placement new机制

参考：[placement new机制 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/228001107)

一般来说，使用new申请空间时，是从系统的“堆”（heap）中分配空间。申请所得的空间的位置是根据当时的内存的实际使用情况决定的。但是，在某些特殊情况下，可能需要在已分配的特定内存创建对象，这就是所谓的“定位放置new”（placement new）操作。

定位放置new操作的语法形式不同于普通的new操作。例如，一般都用如下语句A* p=new A;申请空间，而定位放置new操作则使用如下语句A* p=new (ptr)A;申请空间，其中ptr就是程序员指定的内存首地址。考察如下程序。

```cpp
#include <iostream>
using namespace std;
 
class A
{
public:
	A()
	{
		cout << "A's constructor" << endl;
	}
 
 
	~A()
	{
		cout << "A's destructor" << endl;
	}
	
	void show()
	{
		cout << "num:" << num << endl;
	}
	
private:
	int num;
};
 
int main()
{
	char mem[100];
	mem[0] = 'A';
	mem[1] = '\0';
	mem[2] = '\0';
	mem[3] = '\0';
	cout << (void*)mem << endl;
	A* p = new (mem)A;
	cout << p << endl;
	p->show();
	p->~A();
	getchar();
}
```

1）用定位放置new操作，既可以在栈(stack)上生成对象，也可以在堆（heap）上生成对象。如本例就是在栈上生成一个对象。

（2）使用语句A* p=new (mem) A;定位生成对象时，指针p和数组名mem指向同一片存储区。所以，与其说定位放置new操作是申请空间，还不如说是利用已经申请好的空间，真正的申请空间的工作是在此之前完成的。

（3）使用语句A *p=new (mem) A;定位生成对象时，会自动调用类A的构造函数，但是由于对象的空间不会自动释放（对象实际上是借用别人的空间），所以必须显示的调用类的析构函数，如本例中的p->~A()。

（4）如果有这样一个场景，我们需要大量的申请一块类似的内存空间，然后又释放掉，比如在在一个server中对于客户端的请求，每个客户端的每一次上行数据我们都需要为此申请一块内存，当我们处理完请求给客户端下行回复时释放掉该内存，表面上看者符合c++的内存管理要求，没有什么错误，但是仔细想想很不合理，为什么我们每个请求都要重新申请一块内存呢，要知道每一次内从的申请，系统都要在内存中找到一块合适大小的连续的内存空间，这个过程是很慢的（相对而言)，极端情况下，如果当前系统中有大量的内存碎片，并且我们申请的空间很大，甚至有可能失败。为什么我们不能共用一块我们事先准备好的内存呢？可以的，我们可以使用placement new来构造对象，那么就会在我们指定的内存空间中构造对象。



# c++ new运算符是如何调用构造函数的



原文链接：https://blog.csdn.net/jiang4357291/article/details/104547867



内存申请和对象构造
       
本文内容简短，只为记录下一次思考过程，事情起源于一句话，“new操作符会调用operator new分配内存再调用构造函数构造对象”，但最近再次看到这句话的时候越看越有疑问，怎么样调用构造函数？？于是就带着这个问题顺便看下operator new和placement new的源码加深下印象了。

众所周知构造函数是在实例化一个类对象时调用，通常我们看到的构造函数是下面这样用：

```c++
class A{};

int main()
{
    A inst = A();
    A inst2;
    A *p = new A();
}
```

上述几种调用方式，不管是在栈上还是堆上实例化一个类对象时，共同点都是内存的申请和类的构造是绑定在一起的。可以试想一下，既然new是先调用operator new再调用构造函数，那么这个过程如果我们自己手动调用呢：

```c++
#include <iostream>
using namespace std;

class A{
public:
    A():a(0){cout<<"A() called!"<<endl;}
    int a;
};

int main()
{
    A *p = (A*)::operator new(sizeof(A));// 直接调用operator new，只分配内存
    p->A();
}
```


看起来是没什么问题，但实际结果呢？直接编译出错，报错为：

错误：错误地使用了‘A::A’

即使更改为显示调用，仍然无法编译通过：

```c++
int main()
{
    A *p = (A*)::operator new(sizeof(A));// 直接调用operator new，只分配内存
    p->A::A();
}
```


错误：不能直接调用构造函数‘A::A’

到这里就产生了深深的疑问，构造函数不能被显示调用，那new是怎么调用构造函数的？

placement new
另一方面，我们知道placement new就是在已分配内存上构造对象的，那么placement new是如何只调用构造函数的？

```c++
class A{
public:
    A():a(0){cout<<"A() called!"<<endl;}
    int a;
};

int main()
{
    char buff[4];
    A *p = new(buff) A;// 在buff上调用构造函数
    p->~A();// placement new需要显式析构
    return 0;
}
```


答案很简单
先来看下operator new和placement new的实现：

```c++
operator new:
_GLIBCXX_WEAK_DEFINITION void *
    operator new (std::size_t sz) _GLIBCXX_THROW (std::bad_alloc)
    {
      void *p;

      /* malloc (0) is unpredictable; avoid it.  */
      if (sz == 0)
        sz = 1;
      p = (void *) malloc (sz);
      while (p == 0)
        {
          new_handler handler = std::get_new_handler ();
          if (! handler)
            _GLIBCXX_THROW_OR_ABORT(bad_alloc());
          handler ();
          p = (void *) malloc (sz);
        }

  return p;
}



```


placement new:

```c++
// Default placement versions of operator new.
inline void* operator new(std::size_t, void* __p) _GLIBCXX_USE_NOEXCEPT{ return __p; }
inline void* operator new[](std::size_t, void* __p) _GLIBCXX_USE_NOEXCEPT{ return __p; }
// Default placement versions of operator delete.
inline void operator delete  (void*, void*) _GLIBCXX_USE_NOEXCEPT { }
inline void operator delete[](void*, void*) _GLIBCXX_USE_NOEXCEPT { } 
```

可以看到，operator new只是对malloc的一些封装，而placement new则是什么都没做只是把指针返回。至此也基本可以猜到答案了，答案就是编译器自动调用的，想一下也必然是这样，new是个运算符不是函数，运算符是一种告诉编译器执行特定的数学或者逻辑操作的符号，又何来定义之说。

下面是一些参考：
Thus, for a class type T, a new-expression such as:

```c++
pt = new T;
```


translates more-or-less into something like:

```c++
pt = static_cast(operator new(sizeof(T)));
pt->T();
```


The first statement acquires storage for a T object by calling operator new, and converts the address of that storage from type void * to type T *. The second initializes the storage by applying T’s default constructor. As I mentioned earlier, a C++ compiler won’t let you write this explicit constructor call, but it’s happy to do it for you.