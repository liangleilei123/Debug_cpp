转自百度百科

​    \#pragma once

　　这是一个比较常用的指令,只要在头文件的最开始加入这条指令就能够保证头文件被编译一次
　　#pragma once用来防止某个头文件被多次include，#ifndef，#define，#endif用来防止某个宏被多次定义。
　　#pragma once是编译相关，就是说这个编译系统上能用，但在其他编译系统不一定可以，也就是说移植性差，不过现在基本上已经是每个编译器都有这个定义了。
　　#ifndef，#define，#endif这个是C++语言相关，这是C++语言中的宏定义，通过宏定义避免文件多次编译。所以在所有支持C++语言的编译器上都是有效的，如果写的程序要跨平台，最好使用这种方式
　　#pragma
　　语言符号字符串语言符号字符串是给出特有编译器指令和参量的字符序列。数字符号(#)必须是包含编译指示行中的第一个非空白字符。空白字符可分开数字符号(#)和单词pragma�。
　　作用：
　　为了避免同一个文件被include多次
　　1 #ifndef方式
　　2 #pragma once方式
　　在能够支持这两种方式的编译器上，二者并没有太大的区别，但是两者仍然还是有一些细微的区别。
　　方式一：
　　#ifndef __SOMEFILE_H__
　　#define __SOMEFILE_H__
　　... ... // 一些声明语句
　　#endif
　　方式二：
　　#pragma once
　　... ... // 一些声明语句
　　#ifndef的方式依赖于宏名字不能冲突，这不光可以保证同一个文件不会被包含多次，也能保证内容完全相同的两个文件不会被不小心同时包含。当然，缺点就是如果不同头文件的宏名不小心“撞车”，可能就会导致头文件明明存在，编译器却硬说找不到声明的状况
　　#pragma once则由编译器提供保证：同一个文件不会被包含多次。注意这里所说的“同一个文件”是指物理上的一个文件，而不是指内容相同的两个文件。带来的好处是，你不必再费劲想个宏名了，当然也就不会出现宏名碰撞引发的奇怪问题。对应的缺点就是如果某个头文件有多份拷贝，本方法不能保证他们不被重复包含。当然，相比宏名碰撞引发的“找不到声明”的问题，重复包含更容易被发现并修正。
　　方式一由语言支持所以移植性好，方式二 可以避免名字冲突





## 