## size_t、ptrdiff_t



原文链接：

对于***\*指向同一数组\****arr[5]中的两个指针之差的验证：

   数组如下：ptr = arr;



\-------------------------------------------------------------------------------------------

```
int _tmain(int argc, _TCHAR* argv[])

{

char arr[5] = {1,2,3,4,5};

char *ptr = arr;

printf("%d\n",&ptr[4]-&ptr[0]);

system("PAUSE");

return 0;

}
```

\-------------------------------------------------------------------------------------------

 

运行输出：4

 

更换为字符[数组](https://so.csdn.net/so/search?q=数组&spm=1001.2101.3001.7020)，测试结果一样。

《C和指针》P110 分析如下：两个指针相减的结果的类型为***\*ptrdiff_t\****,它是一种有符号整数类型。***\*减法运算的值为两个指针在内存中的距离\****（以数组元素的长度为单位，而非字节），因为减法运算的结果将除以数组元素类型的长度。所以***\*该结果与数组中存储的元素的类型无关\****。

 

类似的还有如下类型：（点击这里）

**size_t**是unsigned类型，用于指明数组长度或下标，它必须是一个正数，std::size_t.设计size_t就是为了适应***\**多个平台\**\***，其引入增强了程序在不同平台上的可移植性。

**ptrdiff_t**是signed类型，用于存放同一数组中两个指针之间的差距，它可以使负数，std::ptrdiff_t.同上，使用ptrdiff_t来得到***\**独立于平台\**\***的地址差值.

**size_type**是unsigned类型,表示容器中元素长度或者下标，vector<int>::size_type i = 0;

**difference_type**是signed类型,表示迭代器差距，vector<int>:: difference_type = iter1-iter2.

前二者位于标准类库std内，后二者专为STL对象所拥有。

//=====================================================================================

http://blog.csdn.net/yyyzlf/article/details/6209935

C and C++ define a special type for pointer arithmetic, namely ptrdiff_t, which is a typedef of a platform-specific signed integral type. You can use a variable of type ptrdiff_t to store the result of subtracting and adding pointers.For example:

```cpp
#include <stdlib.h>



int main()



{



  int buff[4];



  ptrdiff_t diff = (&buff[3]) - buff; // diff = 3



  diff = buff -(&buff[3]); //  -3



}
```

What are the advantages of using ptrdiff_t? First, the name ptrdiff_t is self-documenting and helps the reader understand that the variable is used in pointer arithmetic exclusively. Secondly, ptrdiff_t is portable: its underlying type may vary across platforms, but you don't need to make changes in the code when porting it.