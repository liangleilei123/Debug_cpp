# C++ allocator类



参考：https://blog.csdn.net/u012069234/article/details/122334616?spm



new有一些灵活上的局限，其中一方面表现在它将内存分配和对象构造组合在了一起。类似的，delete将对象析构和内存释放组合在了一起。

当分配一块大内存时，我们通常计划在这块内存上按需构造对象。在此情况下，我们希望将内存分配和对象构造分离。这意味着我们可以分配大块内存，但只在真正需要时才真正执行对象创建操作。

## allocator类

标准库allocator类定义在头文件memory中，它帮助我们将内存分配和对象构造分离开来。它提供一种类型感知的内存分配方法，它分配的是原始的、未构造的。

为了定义一个allocator对象，我们必须指明这个allocator可以分配的对象类型。当一个allocator对象分配内存时，它会根据给定的对象类型来确定恰当的内存大小和对齐位置：

```c++
allocator<string> alloc;  // 可以分配string的allocator对象
auto const p = alloc.allocate(n);  //分配n个未初始化的string
```



|                     | **标准库allocator类及其算法**                                |
| ------------------- | ------------------------------------------------------------ |
| allocator<T> a      | 定义了一个名为a的allocator对象，它可以为类型为T的对象分配内存 |
| a.allocate(n)       | 分配一段原始的、未构造的内存，保存n个类型为T的对象           |
| a.deallocate(p,n)   | 释放从T*指针p中地址开始的内存，这块内存保存了n个类型为T的对象；p必须是一个先前由allocate返回的指针，且n必须是p创建时所要求的大小，在调用deallocate之前，由用户必须对每个在这块内存中创建的对象调用destory |
| a.construct(p,args) | p必须是一个类型为T*的指针，指向一块原始内存；arg被传递给类型为T的构造函数，用来在p指向的内存中构造一个对象 |
| a.destory(p)        | p为T*类型的指针，此算法对p指向的对象执行析构函数             |

## allocator分配未构造的内存



源码是：

```c++
using value_type = _Ty;

_NODISCARD _CONSTEXPR20_DYNALLOC __declspec(allocator) _Ty* allocate(_CRT_GUARDOVERFLOW const size_t _Count) {
        return static_cast<_Ty*>(_Allocate<_New_alignof<_Ty>>(_Get_size_of_n<sizeof(_Ty)>(_Count)));
    }

#if _HAS_DEPRECATED_ALLOCATOR_MEMBERS
    _CXX17_DEPRECATE_OLD_ALLOCATOR_MEMBERS _NODISCARD __declspec(allocator) _Ty* allocate(
        _CRT_GUARDOVERFLOW const size_t _Count, const void*) {
        return allocate(_Count);
    }
```

返回类型就是value_type*

allocator分配的内存是未构造的，我们按需要在此内存中构造对象。在新标准中，construct成员函数接受一个指针和零个或多个额外参数，在给定位置构造一个元素。额外参数用来初始化构造的对象：

```c++
auto q = p; //q指向最后构造的元素之后的位置
alloc.construct(q++); //*q为空字符串
alloc.construct(q++， 10， 'c'); //*q为cccccccccc
alloc.construct(q++, "hi"); //*q为hi
```

还未构造对象的情况下就是用原始内存是错误的：

```c++
cout << *p << endl;  //正确，使用string的输出运算符
cout << *q << endl;  //灾难，q指向未构造的内存
```

为了使用allocate返回的内存,我们必须用construct构造对象，使用未构造的内存，其行为是未定义的。

当我们用完对象后，必须对每个构造的元素调用destory来销毁他们。函数destory接受一个指针，对指向的对象执行析构函数：

```c++
while(q != p)
    alloc.destory(--q); //释放我们构造的string
```

在循环开始处，q指向最后构造的元素之后的位置。我们在调用destory之前对q进行了递减操作。因此，第一次调用destory时，q指向最后一个构造的元素。最后一步循环中我们destory了第一个构造的元素，随后q将与p相等，循环结束。

一旦元素被销毁后，就可以重新使用这部分内存来保存其他string，也可以将其归还给系统。释放内存通过调用deallocate来完成：

```c++
alloc.deallocate(p, n);
```

我们传递给deallocate的指针不能为空，它必须指向由allocate分配的内存。而且，传递给deallocate的大小参数必须与调用allocate分配内存时候提供的大小参数有一样的值