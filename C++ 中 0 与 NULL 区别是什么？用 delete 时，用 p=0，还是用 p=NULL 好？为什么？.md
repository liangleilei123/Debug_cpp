# C/C++ 中 0 与 NULL 区别是什么？用 delete 时，用 p=0，还是用 p=NULL 好？为什么？



参考：https://www.zhihu.com/question/22203461

作者：蓝色
链接：https://www.zhihu.com/question/22203461/answer/20875324
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



首先呢，要明白一点儿，NULL是一个无类型的东西，而且是一个宏。而宏这个东西，从C++诞生开始，就是C++之父嗤之以鼻的东西，他推崇尽量避免宏。而在他的FAQ中，也有相应的一个关于NULL与0的解释，也谈到了这一点儿。

[Stroustrup: C++ Style and Technique FAQ](https://link.zhihu.com/?target=http%3A//www.stroustrup.com/bs_faq2.html%23null)



在C++标准中，我们可以见到一个词语叫做[null pointer constant](https://www.zhihu.com/search?q=null+pointer+constant&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A20875324})，其实在C++11标准前，是只承认0为null pointer constant的。所以，在C++中，我们也经常能听到一个说法，就是赋予null pointer，应该是使用0，而非NULL。而nullptr pointer constant这个词语在C++11发布后，终于再添了一个成员，就是nullptr。而与NULL本质不同的是，nullptr是有类型的（放了在stddef头文件中），类型是 typdef decltype(nullptr) nullptr_t; 而正是因为是有类型的，这给我们编译器实现nullptr的时候带来了更多细节的考虑，当然也给了使用者更多的保障，所以**如果你的编译器支持nullptr，请一定使用nullptr！**



而nullptr的出现背景，其实是很简单的，C++哲学上来说就是C++之父一直对null pointer没有一个正式的表示感到非常不满，而更工程的来说，就是关于重载这个问题。

```text
void f(void*)
{
}

void f(int)
{
}

int main()
{
    f(0); // what function will be called?
}
```





而引入了nullptr，这个问题就得到了真正解决，会很顺利的调到void f(void*)这个版本。

好了，真的以为nullptr就这样了么? 我前面说过了nullptr是有类型的，叫做nullptr_t，这给我们编译器实现带来了诸多要考虑的东西，不幸的话让我们来举点儿奇葩例子吧！

```cpp
union U
{
    long i;
    nullptr_t t;
};

int main()
{
    U u;
    u.i = 3;
    printf("%ld\n",(long)u.t); // What it is? 0 or 3?
}
```

那么这是应该符合union语意还是nullptr的语意呢？这在标准中是没有说的，我们也为此争论了非常久。当然在我们编译器的实现还是保持了nullptr的语意，结果是0。

而nullptr有类型后，还能做什么呢？那当然就是可以捕获异常了。

```cpp
int main()
{
  try
  {
    throw nullptr;  
  }
  catch(nullptr_t)
  {
 
  } 
}
```

你扔一个NULL试试？看他应该用什么收，正是因为没有类型，所以就要用它的本质类型，比如long什么的来说。你扔一个0试试？那就也不是所谓的空指针类型了，就是要用int什么的来收了。

所以，推崇nullptr是有道理的，我们在编译器实现nullptr的时候考虑了非常非常多的细节，还有很多你们可能一直用不到的情况，我们都要用来测试，目的就是保障开发者的使用。再次那句话，**如果你的编译器支持nullptr，请一定使用nullptr！**



最后再扯一点儿，0在C++是很神奇的东西。比如[纯虚函数](https://www.zhihu.com/search?q=纯虚函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A20875324})为什么是用=0来设置的，不知道有没有同学去考虑过这个问题没有。如果你深刻理解了C++哲学，这应该就是非常简答的问题了。学语言嘛，一定要学到其哲学，你才能知道其之美，其之威力，尤其是C++。