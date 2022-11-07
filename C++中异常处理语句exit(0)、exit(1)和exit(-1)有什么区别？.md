# C++中异常处理语句exit(0)、exit(1)和exit(-1)有什么区别？



版权声明：本文为CSDN博主「李易安」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/melancholyming/article/details/84299946



exit为C++的退出函数，声明于stdlib.h中，对于C++其标准的头文件为cstdlib,声明为
void exit(int value);
exit的功能为，退出当前运行的程序，并将参数value返回给主调进程。
在main中return v;的效果 与exit(v);相同。
exit(1)和exit(-1)
是分别返回1和-1到主调程序。
exit(0)则是返回0。
exit(0)表示程序正常退出，非0表示非正常退出

