# C++学习笔记





## 类

### 封装

封装是保护类的成员不被随意访问的能力。通过把类的实现细节设置为private,我们就能完成类的封装。封装实现了类的接口和实现的分离。封装有两个重要的有点：一是确保用户代码不会无意间破坏封装对象的状态。二是被封装的类的具体实现细节可以随意改变，而无需调整用户级别的代码。

访问说明符：

定义在public说明符之后的成员在整个程序内可被访问，public成员定义类的接口。

定义在private说明符之后的成员可以被类的成员函数访问，但是不能被使用该类的代码访问，private部分封装了类的实现细节。





### 构造函数

coder为类提供了一个构造函数后，编译器将不会自动生成默认的构造函数，如果我们的类需要默认构造函数，必须显示地把它声明出来。我们使用 ` = default`来告诉编译器为我们合成默认的构造函数。如：`Screen() = default`



### 名字查找与类的作用域

如果某个成员的声明使用 了类中尚未出现的名字， 则编译器将会在定义该类的作用域中继续查找。

成员定义中的普通块作用域的名字查找

成员函数中使用的名字按照如下方式解析：
• 首先， 在成员函数内查找该名字的声明。和前面一样， 只有在函数使用之前出现的声明才被考虑。函数的参数列表也算函数内。
• 如果在成员函数内没有找到，则在类内继续查找，这时类的所有成员都可以被考虑。
• 如果类内也没找到该名字的声明， 在成员函数定义之前的作用域内继续查找。

```c++
int height; ／／定义了一个名宇， 稍后将在Screen中使用
class Screen {
public: 
    typedef std: :string: :size_type pos; 
    void dummy_fcn(pos height) { 
		cursor = width * height;		／/哪个height?是那个参数
	}
private:
	pos cursor = O; 
	pos height = 0, width = O;
};
```

当编译器处理dumrny_fcn中的乘法表达式时， 它首先在函数作用域内查找表达式中用到 的名字。 函数的参数位千函数作用域内， 因此dumrny_fcn函数体内用到的名字height 指的是参数声明。**在成员函数内查找名字的声明时，只有在函数使用之前出现的声明才被考虑，所以在dummy_fun函数内使用的height是类之前定义的height**





## 关键字

### const关键字

https://blog.csdn.net/song240948380/article/details/118967350



#### 顶层const和底层const

顶层const可以表示任意的对象是常量。

表示指针本身是个常量，使用时不能改变指针的指向，但可以修改指针所指的对象。

`int i = 0;`

`int *const p1 = &i;		//不能改变p1的值，这是个顶层const`

`const int ci = 42;		//不能改变ci的值，这是个顶层const`



底层const与指针和引用等复合类型的基本类型部分有关。

表示指针所指的对象是一个常量。

当执行对象的拷贝操作时，拷入和拷出的对象必须具有相同的底层const资格 

`const int *p2 = &ci;	//允许改变p2的值，这是个底层const`

对于变量类型的阅读要从右往左阅读。

p2先是一个指针，指向const int 类型，说明这是个底层指针，指向的是个常量对象。



#### 指针和const

第一种：const在'*'的前面，让指针指向一个常量对象，这样可以防止使用**该指针**来修改所指向的值

```c++
int age = 39;
const int * pt = &age;
```

该声明指出，pt**指向的**是一个const int对象，因此不能用pt来修改这个值，但是允许pt指向另一个位置。

**细节**：这个声明只表示对指针pt来说，age是常量，pt不能改变age的值，并不代表age是常量，依旧可以通过变量age来改变age的值。

**扩展：**

- 上述情况是把常规变量赋给了const指针，const指针不能改变变量的值，但变量自身可以改变
- 第二种情况：把const变量赋给const指针，既不能用const指针修改变量的值，变量自身也不能改变
- 第三种情况：把const变量赋给常规指针，**错误**：因为变量本身不能改变，却给指针改变变量的权力，冲突。

第二种：const在'*'的后面，将指针本身声明为常量，这样可以防止改变指针指向的位置。

```c++
int sloth = 3;
const int *ps = &sloth;		//a pointer to const int，
							//不允许通过ps修改sloth的值，但是允许ps指向别的位置
int * const finger = &sloth;//a const pointer to int
							//允许通过finger修改sloth的值，不允许finger指向别的位置
```

​	在最后一个声明中，关键字const的位置与以前不同，这种声明格式使得finger只能指向sloth，但是允许使用finger来修改sloth的值。



第三种：可以声明指向const对象的const指针

```c++
double trouble = 2.0E30;
const double * const stick = &trouble;
```

该声明表示：stick只能指向trouble，而stick不能修改trouble的值。简而言之，stick和*stick都是const

第四种：用指针指向指针，这是两级的间接关系，此时就不能将非const指针指向const指针

```c++
const int **pp2;
//int *p1;		//错误
const int *p1;
const int n = 13;
pp2 = &p1;
*pp2 = &n;
//*p1 = 10;		//错误，
```

尽可能使用const:

将指针参数声明为指向常量数据的指针有两条理由:

- 这样可以避免由于无意间修改数据而导致的编程错误;
- 使用const 使得函数能够处理.const和非const实参，否则将只能接受非const数据。
- 如果条件允许,则应将指针形参声明为指向const的指针。



#### const 参数传递

**const形参和非const形参：**

- 普通形参加不加const限定符对实参没有影响，因为函数是拷贝了一份形参数据在函数内使用。无法修改实参
- 引用形参和指针形参前面没有const限定符时，实参必须是非const的，而前面有const限定符时实参可以是const 也可以是非const。函数是对实参直接操作，没有const的形参时，实参的值是可以改变的。而且非const的形参不能传递常量或const的形参

```c++
#include<iostream>
#include<string>
using namespace std;

void print_str(string &s) {			//string 前加const就没错了
	cout << s << endl;
}

int main() {

	print_str("hello world");			//error：hello world 是常量，不能被非常量类型引用
	
	return 0;

}
```



#### 重载和const形参

- 一个拥有顶层const的形参无法和另一个没有顶层 const的形参区分开来。

```c++
Record lookup(Phone);
Record lookup(const Phone); // 重复声明了 Record lookup(Phone)
Record lookup(Phone*);
Record lookup(Phone* const); // 重复声明了 Record lookup(Phone*)
```



- 如果形参是某种类型的指针或引用，则通过区分其指向的是常量对象还是非常量对象可以实现函数重载，此时的const是底层的。

```c++
//对于接受引用或指针的函数来说，对象 是常量还是非常量对应的形参不同
Record lookup(Account&); // function that takes a reference to Account
Record lookup(const Account&); // new function that takes a const reference

Record lookup(Account*); // new function, takes a pointer to Account
Record lookup(const Account*); // new function, takes a pointer to const
```



#### const map 和const_iterator搭配

```c++
//在显示单词之前，允许用户查询某个单词是否出现在文本文件中
void user_select(const map<string,int> &map_words) {
	//提示用户输入要查找的单词
	string user_word;
	cout << "please return the word that you want to select(q to quit):\n";
	cin >> user_word;
	while (user_word.size() && user_word != "q") {
		map<string, int>::const_iterator map_it = map_words.begin();		//const map 和const_iterator搭配
		if ((map_it = map_words.find(user_word)) != map_words.end()) {
			cout << "Found!\n" << map_it->first
				<< "\tCount: " << map_it->second << endl;
		}
		else {
			cout << "Not Found!\n";
		}
		cout << "Another word?(q to quit):";
		cin >> user_word;
	}
	
	
}
```



#### 类相关CONST

https://blog.csdn.net/eric_jo/article/details/4138548

https://blog.csdn.net/piaopiaopiaopiaopiao/article/details/12254277?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&utm_relevant_index=2



摘要：

- const reference 类对象 做参数传递，不能改变类成员数据，不能调用类中non_const成员函数

成员函数：

- 用const修饰类的成员函数，该函数不能修改类成员数据，不能调用non_const成员函数
-  const对象只能访问const成员函数,而const成员函数可以被其他成员函数调用
-  const对象的成员是不可修改的,然而const对象通过指针维护的对象却是可以修改的.
- const成员函数不可以修改对象的数据,不管对象是否具有const性质.它在编译时,以是否修改成员数据为依据,进行检查.

数据成员：

- 加上mutable修饰符的数据成员,对于任何情况下通过任何手段都可修改,自然此时的const成员函数是可以修改它的
- 非const成员函数可以访问非const对象的非const数据成员、const数据成员，但不可以访问const对象的任意数据成员
- const成员函数可以访问非const对象的非const数据成员、const数据成员，也可以访问const对象内的所有数据成员





##### const修饰成员变量

const修饰类的成员函数，表示成员常量，不能被修改，同时它只能在初始化列表中赋值。

```c++
    class A
​    {
​        …
​        const int nValue;         //成员常量不能被修改
​        …
​        A(int x): nValue(x) { } ; //只能在初始化列表中赋值
​     }


```

##### const修饰成员函数

const修饰类的成员函数，则该成员函数不能修改类中任何非const成员函数。一般写在函数的最后来修饰。

```c++
  class A
    {
        …
       void function()const; //常成员函数, 它不改变对象的成员变量.                        

							//也不能调用类中任何非const成员函数。
}
```

对于const类对象/指针/引用，只能调用类的const成员函数，因此，const修饰成员函数的最重要作用就是限制对于const对象的使用。

- const成员函数不被允许修改它所在对象的任何一个数据成员。

- const成员函数能够访问对象的const成员，而其他成员函数不可以。

 

```C++
class Person {

public:
	Person(int no = 1,int age = 20,int number = 123)
	:m_no(no),m_age(age),m_number(number){

	}

	//以下是const_member function
	int name() const {return m_no;}
	int age() const { return m_age; }
	int number() const {return m_number;}

	//以下是non_member
	int age_plus (int &m) const{
		m++;
		//m_age += 1;		//ERROR:const修饰的成员函数不能改变成员数据的值
	}

public:
	int m_no;
	int m_age = 20;
	int m_number;
	
	int text_data;
};

int sum(const Person &P1,const Person &p2) {

	//测试通过sum改变const & 参数的值
	//P1.m_age += 2;			//不能通过sum改变类内的变量
	cout << P1.m_age;
	//P1.age_plus();				//不能调用:const reference class参数不能调用公开接收中的non_const成分
	return 0;
}
```

##### const修饰类对象/对象指针/对象引用

- const修饰类对象表示该对象为常量对象，其中的任何成员都不能被修改。对于对象指针和对象引用也是一样。

- const修饰的对象，该对象的任何非const成员函数都不能被调用，因为任何非const成员函数会有修改成员变量的企图。
  例如：

```c++
class AAA
{
    void func1();
void func2() const;
}
const AAA aObj;
aObj.func1(); ×
aObj.func2(); 正确

const AAA* aObj = new AAA();
aObj-> func1(); ×
aObj-> func2(); 正确

```

##### 常量引用可以绑定非常量的对象、字面值和一般表达式

https://blog.csdn.net/Colsum/article/details/79095462

在c++语言中，除两种例外情况，其他引用的类型都要和与之绑定的对象严格匹配，如int型的引用只能绑定int型的对象；并且引用不能直接与字面值常量或表达式结果绑定。

    其中一种例外情况是：初始化常量引用时，允许用任意表达式作为初始值，只要该表达式的结果能转换成引用的类型即可。允许为一个常量引用绑定非常量的对象、字面值，甚至是个一般表达式。例如：


    // 以下例子来自《Primer c++ 第五版》
    int i = 42;
    const int &r1 = i;      // 正确
    const int &r2 = 42;     // 正确
    const int &r3 = r1 * 2; // 正确
    
    接下来看一个更复杂的例子，并通过这个例子来讲解为什么常量引用是个例外。
    
    double dval = 3.14;
    // int &a = dval;    // 编译错误，因为普通引用的类型要与对象类型一致
    const int &b = dval; // 编译正确
    
    c++的自动类型转换机制中，当用一个double去初始化int时，会舍弃掉小数转换为int。在上面的例子中，编译后的代码实际是这样的：
    
    double dval = 3.14;
    const int temp = dval; // 由double生成了一个临时的整形常量
    const int &b = temp;   // 让b绑定这个临时量
    
    代码中的改变当然是由编译器完成的。
    
    在这种情况下，引用绑定的是一个临时量对象而不是dval本身。临时量对象就是：当编译器需要一个空间来暂存表达式的求值结果时，临时创建的一个未命名的对象。
    
    显然，c++认为，常量引用可以绑定这个临时量，而普通引用就不能绑定这个临时量。
    
    因为c++认为，使用普通引用绑定一个对象，就是为了能通过引用对这个对象做改变。如果普通引用绑定的是一个临时量而不是对象本身，那么改变的是临时量而不是希望改变的那个对象，这种改变是无意义的。所以规定普通引用不能绑定到临时量上。
    
    那么为什么常量引用就可以呢，因为常量是不能改变的。也就是说，不能通过常量引用去改变对象，那么绑定的是临时量还是对象都无所谓了，反正都不能做改变也就不存在改变无意义的情况。
    
    所以常量引用可以绑定临时量，也就可以绑定非常量的对象、字面值，甚至是一般表达式，并且不用必须类型一致。

##### 非常量引用不能绑定字面量





### mutable

我们希望能修改类的某个数据成员，即使实在一个const成员函数内。可以通过在变量声明中加入mutable关键字使其称为可变数据成员。一个可变数据成员永远不会是const，即使他是const对象的成员。

```C++
class Screen{
public:
	void some_number() const;
private:
	mutable size_t access_ctr;		//即使在一个const对象内也能被修改
};

void Screen::some_number() const{
    ++ access_ctr;
}
```







### Typedef

https://zhuanlan.zhihu.com/p/413574268

#### Typedef定义成员函数指针

```c++
#include <iostream>
 
class foo
  {
public:
  int g (int x, int y) { return x + y ; }
  } ;
 
typedef int (foo::*memberf_pointer)(int, int);//将memberf_pointer声明为指向foo的成员函数指针
 
int main()
  {
  foo f ;	//声明类
  foo *p;	//声明类指针
  memberf_pointer mp = &foo::g ;		//定义，初始化赋值操作
  std::cout << (f.*mp) (5, 8) << std::endl ;	//调用成员函数， .* 是指向成员选择的指针，针对class object工作
  std::cout << (p->*mp) (5, 8) << std::endl ;	//->*是类指针指向成员选择指针的运算符
  }
```



#### typedef int P();

```c++

#include <iostream>
#include <stdio.h>
#include <string>

typedef int P(); // 简单的,定义一个名为P的函数指针类型，参数为空，返回值为int类型
//P(Q);// 相当于声明Q是一个返回值为int，参数为空的返回类型。主要用于类内函数声明。
typedef void Q(int* p, const std::string& s1, const std::string& s2, size_t size, bool is_true); // 复杂的
class X {
public:
    P(eat_shit); // 等价于声明`int eat_shit();`
    Q(bullshit); // 等价于声明`void bullshit(int *p, const string& s1, const string& s2, size_t size, bool is_true);`
};

int main() {
    X xx;
    printf("shit ret: %d\n", xx.eat_shit());
    int a[] = { 1, 3, 4, 5, 7 };
    xx.bullshit(a, "foo", "bar", sizeof(a) / sizeof(int), true);
}

int X::eat_shit() {
    return 888;
}

void X::bullshit(int* p, const std::string& s1, const std::string& s2, size_t size, bool is_true) {
    std::cout << "s1: " << s1 << ", s2: " << s2 << ", size: " << size << std::endl;
    printf("elems:\n");
    for (unsigned int i = 0; i < size; i++) {
        printf("%d %s", *p++, (i == size - 1) ? "" : ",");
    }
    printf("\n");
}
```



#### 别名声明

https://blog.csdn.net/csxiaoshui/article/details/78038799

新标准规定的方法：

`using SI = Sales_item；	//SI是Sales_item的同义词`

用typedef定义类型别名

https://blog.csdn.net/LaterEqualsNever1024/article/details/74835721?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.pc_relevant_default&utm_relevant_index=1

![img](https://img-blog.csdn.net/20170708154627340?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTGF0ZXJFcXVhbHNOZXZlcjEwMjQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)







### 枚举enum

https://www.runoob.com/w3cnote/cpp-enum-intro.html

`enum spectrum{red,orange,yellow,green,blue,violet,indigo,ultraviolet};`

这条语句完成两项工作：

- 让spectrum成为新类型的名称；spectrum被称为枚举（enumeration）,就像struct变成被称为结构一样
- 将red、 orange、 yellow等作为符号常量,它们对应整数值0~7。这些常量叫作枚举量(enumerator)。

在默认情况下，将整数值赋给枚举量，第一个枚举量的值为0，第二个枚举量的值为1，依次类推。可以通过显式地指定整数值来覆盖默认值。

**操作：**

1. 可以用枚举名来声明这种类型的变量：`spectrum band;`
2. 在不进行强制类型转换的情况下，只能将定义枚举时使用的枚举量赋给这种枚举的变量

```c++
band = blue; //合法   
band = 2000; //不合法
band = orange + red;	//合法
++band;					//不合法
```

​	3.枚举量是整型，可被提升为int类型，但int类型不能自动转换为枚举类型:

```c++
int color = red;	//valid
band = 3;		//invalid（无效）
color = 3 + red;
```

**重要提示：**

枚举常量代表该枚举类型的变量可能取的值，编译系统为每个枚举常量指定一个整数值，默认状态下，这个整数就是所列举元素的序号，序号从0开始。 可以在定义枚举类型时为部分或全部枚举常量指定整数值，在指定值之前的枚举常量仍按默认方式取值，而指定值之后的枚举常量按依次加1的原则取值。 各枚举常量的值可以重复。例如：

```c++
enum fruit_set {apple, orange, banana=1, peach, grape}
//枚举常量apple=0,orange=1, banana=1,peach=2,grape=3。
enum week {Sun=7, Mon=1, Tue, Wed, Thu, Fri, Sat};
//枚举常量Sun,Mon,Tue,Wed,Thu,Fri,Sat的值分别为7、1、2、3、4、5、6。
```

枚举常量只能以标识符形式表示，而不能是整型、字符型等文字常量。例如，以下定义非法：

```c++
enum letter_set {'a','d','F','s','T'}; //枚举常量不能是字符常量
enum year_set{2000,2001,2002,2003,2004,2005}; //枚举常量不能是整型常量
```

可改为以下形式则定义合法：

```c++
enum letter_set {a, d, F, s, T};
enum year_set{y2000, y2001, y2002, y2003, y2004, y2005};
```



### Static

#### 静态类成员函数

https://www.runoob.com/w3cnote/cpp-static-usage.html

可以将类成员函数声明为静态，

- 由于静态成员函数不与特定的对象相关联，因此只能使用静态数据成员
- 静态数据成员在使用前必须在类外定义

```c++
//.h文件
#include<string>

using namespace std;

class globalWrapper {
public:
	static string program_name() { return _program_name; }

	static void program_name(const string& npn) {
		_program_name = npn;
	}
    
    //测试非静态成员函数
    void print()  {cout<< "non_static_member"<<endl;}



private:
	static string _program_name ;
};

string globalWrapper::_program_name = " ";			//静态数据成员使用前必须定义，且必须再类外定义

//.cpp文件
#include<iostream>
#include"4_3.h"

int main() {

	
	globalWrapper glo;
	glo.program_name("4_3cpp");					//静态成员函数可以通过对象访问
	
	cout << globalWrapper::program_name() << endl;	//也可以通过类名和作用域运算符调用
    
    //非静态成员函数
	glo.print();
	//globalWrapper::print();			//ERROR:非静态成员引用必须与特定对象相对

	return 0;

}
```



### extern

https://blog.csdn.net/fengbingchun/article/details/78941738?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2.pc_relevant_default&utm_relevant_index=5



### reverse_iterator反向迭代器

http://c.biancheng.net/view/7274.html

#### C++ STL反向迭代器的创建

reverse_iterator 模板类中共提供了 3 种创建反向迭代器的方法，这里以 vector<int> 容器的随机访问迭代器作为基础迭代器为例。

 \1) 调用该类的默认构造方法，即可创建了一个不指向任何对象的反向迭代器，例如：

```
std::reverse_iterator<std::vector<int>::iterator> my_reiter;
```

由此，我们就创建好了一个没有指向任何对象的 my_reiter 反向迭代器。

 \2) 当然，在创建反向迭代器的时候，我们可以直接将一个基础迭代器作为参数传递给新建的反向迭代器。例如：

```
//创建并初始化一个 myvector 容器std::vector<int> myvector{1,2,3,4,5};//创建并初始化 my_reiter 迭代器std::reverse_iterator<std::vector<int>::iterator> my_reiter(myvector.end());
```

我们知道，反向迭代器是通过操纵内部的基础迭代器实现逆向遍历的，但是反向迭代器的指向和底层基础迭代器的指向并不相同。以上面创建的  my_reiter 为例，其内部的基础迭代器指向的是 myvector 容器中元素 5 之后的位置，但是 my_reiter 指向的却是元素  5。

 也就是说，反向迭代器的指向和其底层基础迭代器的指向具有这样的关系，即反向迭代器的指向总是距离基础迭代器偏左 1 个位置；反之，基础迭代器的指向总是距离反向迭代器偏右 1 个位置处。它们的关系如图 1 所示。


 ![img](http://c.biancheng.net/uploads/allimg/200303/2-200303101159441.gif)
 图 1 反向迭代器和基础迭代器的关系

> 其中，begin 和 end 表示基础迭代器，r(begin) 和 r(end) 分别表示有 begin 和 end 获得的反向迭代器。


 \3) 除了第 2 种初始化方式之外，reverse_iterator 模板类还提供了一个复制（拷贝）构造函数，可以实现直接将一个反向迭代器复制给新建的反向迭代器。比如：

```
//创建并初始化一个 vector 容器std::vector<int> myvector{1,2,3,4,5};//调用复制构造函数初始化反向迭代器的 2 种方式std::reverse_iterator<std::vector<int>::iterator> my_reiter(myvector.rbegin());//std::reverse_iterator<std::vector<int>::iterator> my_reiter = myvector.rbegin();
```

由此，my_reiter 反向迭代器指向的就是 myvector 容器中最后一个元素（也就是 5）之后的位置。



### size_type

https://blog.csdn.net/u012246313/article/details/44537757

　string::size_type从本质上来说，是一个整型数。关键是由于机器的环境，它的长度有可能不同。 例如：我们在使用  string::find的函数的时候，它返回的类型就是 string::size_type类型。而当find找不到所要找的字符的时候，它返回的是 npos的值，这个值是与size_type相关的。假如，你是用 string s; int rc = s.find(.....);  然后判断，if ( rc == string::npos )  这样在不同的机器平台上表现就不一样了。如果，你的平台的string::size_type的长度正好和int相匹配，那么这个判断会侥幸正确。但换成另外的平台，有可能 string::size_type的类型是64位长度的，那么判断就完全不正确了。 所以，正确的应该是： string::size_type  rc = s.find(.....); 这个时候使用 if ( rc == string::npos )就回正确了。

 我们为什么不适用int变量来保存string的size呢？

  使用int变量的问题是：有些机器上的int变量的表示范围太小，甚至无法存储实际并不长的string对象。如在有16位int型的机器上，int类型变量最大只能表示32767个字符的string对象。而能容纳一个文件内容的string对象轻易就能超过这个数字，因此，为了避免溢出，保存一个string对象的size的最安全的方法就是使用标准库类型string：：size_type().



### decltype自动推导类型

http://c.biancheng.net/view/7151.html





## `IO`

### 文件IO

```C++
	ifstream in_file("input_file.txt");			//默认打开模式是"in"，只能打开已存在的文件，文件不存在，打开错误
	ofstream out_file("output_file.txt");		//默认打开模式是"out|trunc",打开文件，接收程序写入时清除原有的文件内容
	cerr << "ERROR:unable to open the necessary file.\n"; //cerr不经过缓冲而直接输出，一般用于迅速输出出错信息，是标准错误，
															//默认情况下被关联到标准输出流，但它不被缓冲，
															//也就说错误消息可以直接发送到显示器，而无需等到缓冲区或者新的换行符时，才被显示。
//直接从文件读，然后放入map中，C++默认读取文件是，遇到空格或换行符就停止读取
	string str_word;
	while (in_file >> str_word) {
		if (except_words.count(str_word)) {
			continue;
		}
		words[str_word]++;
	}

```



### cin 判断输入结束（读取结束）

http://c.biancheng.net/view/277.html

windows 第一个字符是 ctrl+z +enter

UNIX第一个字符是 ctrl+D+enter



### ostream 

https://blog.csdn.net/sinat_36219858/article/details/80380851

把ostream引用放在函数参数中有什么用，为啥不直接用cout?

- 引用，是因为无法直接复制一个ostream对象
- cout是标准输出iostream的一个对象，用ostream这样不仅仅可以标准输出iostream对象，也可以是文件输出类ofstream 或者ostringstream的一个对象！



## STL std::copy()

https://blog.csdn.net/a_ran/article/details/17385911

### copy()+iostream iterator







## 容器vector常用接口

https://blog.csdn.net/qq_41071068/article/details/101421847



### vector::back()

https://blog.csdn.net/cumt951045/article/details/107796082

函数原型：reference back();

​					const_reference back() const;

参数：无

返回值：返回最后一个元素的引用



### vector::max_size()

参数：无

返回值：size_type返回向量的最大大小，为无符号整数类型











## 运算符

### for循环中++、--

https://www.codenong.com/484462/

这里以++为例，--同理

`for(int i = 0;i < 5;i++)`

内在逻辑，++运算符的前缀和后缀的区别体现在返回值上，而在for语句中并没有用到返回值，在执行i++或++i语句后i都会加1。所以在此处前缀和后缀区别不大，但是提倡用前缀。

当for循环中用到i值时，再根据情况用前缀或后缀



### 作用域运算符::

:是运算符中等级最高的，它分为三种：全局作用域符，类作用域符，命名空间作用域符

##全局作用##

全局作用域符号：当全局变量在局部函数中与其中某个变量重名，那么就可以用::来区分如：
　　char ch; //全局变量
　　void sleep（）
　　{undefined
　　char ch; //局部变量
　　ch(局部变量) = ch(局部变量) *ch(局部变量) ;
　　::ch(全局变量) =::ch(全局变量) *ch(局部变量);
　　}

##类作用域符号##

类作用域符号::的前面一般是类名称，后面一般是该类的成员名称，C++为了避免不同的类有名称相同的成员而采用作用域的方式进行区分。
　　例如A,B表示两个类，在A,B中都有成员member，那么
　　A::member就表示类A中的成员member
　　B::member就表示类B中的成员member
　　
##命名空间##

"::"是作用域限定符或者称作用域运算符或者作用域操作符（scope operator），例如命名空间。
"::"作用：namespace::name
:: 的另一种用法
直接用在全局函数前，表示是全局函数。当类的成员函数跟类外的一个全局函数同名时，在类内定义的时候，用此函数名默认调用的是本身的成员函数；如果要调用同名的全局函数时，就必须打上::以示区别。
————————————————
版权声明：本文为CSDN博主「雪舞飞影」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/dongxianfei/article/details/53883878



### reference to pointer *& 指针的引用

https://zhuanlan.zhihu.com/p/139543762

```c++
int v = 1;
int *p = &v;
int* &rp = p;		//*rp相当于*p的别名，修改*rp相当于修改*p
```

&是引用符号。引用不产生副本，而是给变量起别名。对原变量操作就是对原变量操作。

指针引用的作用：

- 在参数传递时，单用指针传递指针在函数体内局部的修改指针指向的对象，指针的引用能够全局修改指针指向的对象

```C++
#include<iostream>

using namespace std;

int main() {

	int a = 1;
	int* p = &a;

	int b = 2;
	int* p2 = &b;

	cout << "*p的地址为：" << p<< endl
		<< "*p的值为：" << *p << endl;

	cout << "*p2的地址为：" << p2 << endl
		<< "*p2的值为：" << *p2 << endl;

	int* &rp = p;	//
	cout << "*rp的地址为：" << rp << endl
		<< "*rp的值为：" << *rp << endl;				//和*p的数据完全一样

	rp = p2;		//修改rp的指向
	cout << "*rp的地址为：" << rp << endl
		<< "*rp的值为：" << *rp << endl;				//和p2的数据一样

	cout << "修改rp的指向后,p的值\n";
	cout << "*p的地址为：" << p << endl
		<< "*p的值为：" << *p << endl;					//和p2的数据一样，相当于*p指向*p2
	
	return 0;
}
```

```C++
//测试reterence to pointer全局的修改变量

#include <stdio.h>

//void swap(int* p1, int* p2) {			//不用*&，p1和p2的交换仅限于swap中
void swap(int* &p1, int* &p2) {			//可以全局的改变*p1和*p2的数据
    
    int* temp = p1;
    p1 = p2;
    p2 = temp;
    printf("交换中：a=%d,b=%d \n", *p1, *p2);
    printf("交换中(地址)：p1=%d \n", p1);
    printf("交换中(地址)：p2=%d \n", p2);
}

int main() {

    int a = 1, b = 3;
    int* p1 = &a, * p2 = &b;

    // 交换前
    printf("交换前：a=%d,b=%d \n", *p1, *p2);
    printf("交换前(地址)：p1=%d \n", p1);
    printf("交换前(地址)：p2=%d \n", p2);
    // 交换中
    swap(p1, p2);
    // 交换后
    printf("交换后：a=%d,b=%d \n", *p1, *p2);
    printf("交换后(地址)：p1=%d \n", p1);
    printf("交换后(地址)：p2=%d \n", p2);
    return 0;

}

```





### 运算符重载的作用

[运算符](https://so.csdn.net/so/search?q=运算符&spm=1001.2101.3001.7020)重载是为了解决类对象之间的运算的，通常的运算符只用于算术运算，如常量int之间，因为编译器已经定义了；而一个类两个对象之间成员进行运算必须重新定义，让编译器在遇到对象运算时能按我们要求的进行运算，这就是运算符重载的意义，即重定义运算符，因此你可以看到，运算符重载就是为[类对象](https://so.csdn.net/so/search?q=类对象&spm=1001.2101.3001.7020)服务的，那么两个对象的成员进行运算那必须先获得对象本身啦，所以运算符重载参数必须含有类指针或引用，这是主要客户。 

   运算符重载的声明[operator](https://so.csdn.net/so/search?q=operator&spm=1001.2101.3001.7020) 关键字告诉  编译器，它是一个运算符重载，后面是相关运算符的符号，如+。返回类型是在使用这个运算符时获得的类型。对于这个+运算符重载，返回类型与包含类一样，但这种情况并不是必需的。两个参数就是要操作的对象。对于二元运算符（带两个参数），如+和－运算符，第一个参数是放在运算符左边的值，第二个参数是放在运算符右边的值。





## 变量



### 常量和字面值常量

https://www.233tw.com/cpp/110115

常量是添加了const的量，字面值常量是可以直接写出来的量，形如 42 ‘a’ "Hello World" 



### 变量初始化

如果定义变量时没有指定初值，则变量被默认初始化(default initialized)，此时变量被赋予了“默认值”。默认值到底是什么由变量类型决定，同时定义变量的位置也会对此有影响。
如果是内置类型的变量未被显式初始化，它的值由定义的位置决定。定义于任何函数体之外的变量被初始化为0。一种例外情况是，定义在函数体内部的内置类型变量将不被初始化(uninitialized)。一个未被初始化的内置类型变量的值是未定义的，如果试图拷贝或以其他形式访问此类值将引发错误。
每个类各自决定其初始化对象的方式。而且，是否允许不经初始化就定义对象也由类自己决定。如果类允许这种行为，它将决定对象的初始值到底是什么。

```C++
#include<iostream>

using namespace std;

int over_non_def;		//定义于任何函数体外的变量，会被自动初始化为0

int main() {

	int non_def;
	int def = 0;

	//函数体内未初始化的变量不能被输出
	//cout << non_def << endl;	//WARNING:使用未初始化的内存“non_def”。	

	//函数体内未初始化的变量不能给其他变量赋值
	//def = non_def;
	//cout << def << endl;

	cout << over_non_def << endl;		//输出结果为0


	return 0;
}
```





### 为什么C++内置类型的局部变量不能默认初始化

https://blog.csdn.net/qq_36946274/article/details/80607439

全局变量，存于静态区（全局区），m是局部变量，存于栈区，局部变量不能默认初始化m是因为它在栈上，全局变量区可以统一清零，但是栈上若加了清零操作，会使得函数调用等操作变得更加缓慢，因此编译器取消了此功能。



### 默认初始化和值初始化

https://blog.csdn.net/J_H_C/article/details/83589282

默认初始化：如果定义变量时没有指定初值，则变量被默认初始化。其初始值和变量的类型以及变量定义的位置相关。默认初始化类对象和默认初始化内置类型变量有所不同。



值初始化：就是用数值初始化变量。如果没有给定一个初始值，就会根据变量或[类对象](https://so.csdn.net/so/search?q=类对象&spm=1001.2101.3001.7020)的类型提供一个初始值。对于int类型其值初始化后的值为0。



## 数组



### 数组的初始化

```C++
//测试数组的初始化、
	//char a[6] = "asdfgh";		//错误：没有空间可以存放空字符
	char a[6] = "asdfg";		//字符串结尾处默认有一个空字符
	char a2[6] = { 'a','s','d','f','g','h' };	//没有空字符
	//[d]，d是数组的维度，但是数组的下表是从0开始的，d是几，数组中就有几个元素，
	//用字符串初始化数组时要注意字符串结尾有默认的空字符
```

### 用for循环修改数组的值

```C++
//用for循环修改数组的值

	//范围for语句，声明与数组元素相关联的变量时用引用
	for (auto &r :a) {
		r += 2;			//r是char类型，用引用相当于给数组内对阿盈的变量取别名
	}

	//传统for语句本身就是用指针对数组元素进行操作的，指针可以间接的访问他所指的对象，所以通过指针可以修改他所指对象的值
	/*for (int i = 0; i < 7; i++) {
		a[i] += 2;
	}*/

	for (auto beg = begin(a), aend = end(a); beg != aend; beg++) {		//C++标准库函数begin返回指向数组的首元素
																		//end返回指向数组尾元素的下一个元素的指针,
																		// 两个函数定义在iterator头文件中
		//beg += 2;
		//cout << *beg << endl;
		*beg +=2;
		//auto& r = *beg;
		//r += 2;
	}

	for (auto r : a) {
		cout << r;
	}
```











## 异常处理



```c++
//编写一段程序，从标准输入读取两个整数，输出第一个数除以第二个数的结果。当第二个数是0时抛出异常。
//使用try语句块捕获异常。catch子句应该为用户输出一天提示信息，询问其是否输入新数并重新执行try语句块的内容。
#include<iostream>
//#include<stdexcept>

using std::cin;	using std::cout; using std::endl; using std::runtime_error;

int main() {

	cout << "输入两个整数:" << endl;
	int num1 = 0;
	int num2 = 0;

	//异常检测并处理
	while (cin >> num1 >> num2) {
		try {

			if (num2 == 0) {
				throw runtime_error("elem2不能是 0 ");
				//cout << num1;		//测试是不是在throw语句的下一条语句返回异常
			}
		}
		catch (runtime_error) {
			//提醒用户除数不能为0
			cout << "除数不能为0!" << endl;
			cout << "Try Again? Enter y or n" << endl;
			char c = ' ';
			cin >> c;
			if (!cin || c == 'n') {
				break;
			}

		}
	}
	
	cout << num1 / num2;

	return 0;
}
```

