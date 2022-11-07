# set_new_handler(0)是什么意思？



参考https://blog.csdn.net/qq_45254369/article/details/124774054



出自《STL源码剖析》第45页中有一行代码`set_new_handler(0)；`

```c++
	inline T* _allocate(ptrdiff_t size, T*)
	{
		std::set_new_handler(0);
		T* tmp = (T*)(::operator new((size_t)(size * sizeof(T))));
		if (tmp == 0)
		{
			std::cerr << "out of menory" << std::endl;
		}
		return tmp;
	}

```

首先说一下C++对内存分配的原理。如果程序员决定用new operator向计算机申请一块内存，那么就可能会遇到内存不够的情况。一旦内存不够申请失败，那么默认情况下C++会抛出std::bad_alloc异常。但是如果你不想让它抛出异常，而是想自己写一个程序来处理内存不够的情况，那么你就可以用set_new_handler(new_handler)，把new_handler指向你写的内存不够的处理程序。这样内存不够了的话C++就会去调用你写的内存不够处理程序，然后再做后续处理。如果你写set_new_handler(0)也就是set_new_handler(nullptr)，实际上就是强制C++认为你没有自定义的内存不够处理程序（因为指针是0嘛），所以说当内存不够的时候，C++就会直接抛出std:bad_alloc异常。

因为在这个例子中是一个模板函数，所以说谁也不知道用户在模板实例化的时候前面有没有给set_new_handler指定什么自定义的内存不够处理函数。所以这个用set_new_handler(0)，就是为了强制C++在内存不够的时候抛出std:bad_alloc，而不是去执行什么其他自定义的内存不够处理程序。所以这里用set_new_handler(0)不是没有用，而是强制C++在内存不够的时候一定要抛出std:bad_alloc异常

总结
set_new_handler(0)主要是为了卸载目前的内存分配异常处理函数，这样就会导致一旦分配内存失败，C++就会强制性抛出std:bad_alloc异常，而不是跑到处理某个异常处理函数去处理。