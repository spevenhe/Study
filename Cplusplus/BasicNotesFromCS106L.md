# 定义
![image](https://user-images.githubusercontent.com/42630862/166891479-ed3b811e-27d5-47df-a506-6468e26d03ef.png)
![image](https://user-images.githubusercontent.com/42630862/166891499-698474a6-40ee-4d35-8f2a-5d8250c7b85e.png)
![image](https://user-images.githubusercontent.com/42630862/166893068-426f9514-15a2-4573-8f31-1bb02b7a7ecd.png)

# 引用
![image](https://user-images.githubusercontent.com/42630862/166898623-daeced22-7c20-452f-9415-4980575c536b.png)

![image](https://user-images.githubusercontent.com/42630862/166898514-9b39db11-ba4a-4660-8697-704d15b29c44.png)
![image](https://user-images.githubusercontent.com/42630862/166898546-124eb547-321c-40d5-87c3-adb3502b8cb5.png)

# const
![image](https://user-images.githubusercontent.com/42630862/166900441-1df6cf57-cc08-4af5-97f5-2944cc615117.png)
![image](https://user-images.githubusercontent.com/42630862/166900546-bf93dce6-27e5-4dc4-a758-1712004444f6.png)
![image](https://user-images.githubusercontent.com/42630862/166900734-991ed09d-5e72-4062-b1fe-f5224f574482.png)
![image](https://user-images.githubusercontent.com/42630862/166900755-05ccedd1-1c0e-4e04-a284-2ce5356d730e.png)

# 迭代器 iterator
是一个抽象概念，理解为指向数据的指针
主要作用 在于为所有的 container提供标准化的函数 

![image](https://user-images.githubusercontent.com/42630862/167169570-901b75f7-394f-4966-8e99-1dd13e67189c.png)
![image](https://user-images.githubusercontent.com/42630862/167300418-1c674c39-4146-44e9-934a-c8f59619e220.png)
![image](https://user-images.githubusercontent.com/42630862/167300980-4a92c3f7-ca9b-4f8d-9be8-0822842bf1cf.png)
![image](https://user-images.githubusercontent.com/42630862/167301253-fd2a4bd0-2c53-4663-8649-bd4c0526bdef.png)
![image](https://user-images.githubusercontent.com/42630862/167301363-8f386661-88c0-4c1e-967e-377812de78eb.png)
![image](https://user-images.githubusercontent.com/42630862/167301389-8e9580b7-3999-4f61-ae76-40d82edc6a96.png)
![image](https://user-images.githubusercontent.com/42630862/167301423-877096b9-7c52-4837-b8f1-6184f750d7ea.png)
![image](https://user-images.githubusercontent.com/42630862/167301453-d357754e-6f29-4f8a-becd-8cf9e52666a2.png)

# class
![image](https://user-images.githubusercontent.com/42630862/167444688-334ac0dc-204d-4eb1-9f3a-524f8510396b.png)

![image](https://user-images.githubusercontent.com/42630862/167444644-be641b64-5982-47d1-8292-e93bf61a1e33.png)

“头文件中的 #ifndef/#define/#endif 防止该头文件被重复引用”

其实“被重复引用”是指一个头文件在同一个cpp文件中被include了多次，这种错误常常是由于include嵌套造成的。比如：存在a.h文件#include "c.h"而此时b.cpp文件导入了#include "a.h" 和#include "c.h"此时就会造成c.h重复引用。



头文件被重复引用引起的后果：

有些头文件重复引用只是增加了编译工作的工作量，不会引起太大的问题，仅仅是编译效率低一些，但是对于大工程而言编译效率低下那将是一件多么痛苦的事情。
有些头文件重复包含，会引起错误，比如在头文件中定义了全局变量(虽然这种方式不被推荐，但确实是C规范允许的)这种会引起重复定义。



    是不是所有的头文件中都要加入#ifndef/#define/#endif 这些代码？

    答案：不是一定要加，但是不管怎样，用#ifnde xxx #define xxx #endif或者其他方式避免头文件重复包含，只有好处没有坏处。个人觉得培养一个好的编程习惯是学习编程的一个重要分支。



    下面给一个#ifndef/#define/#endif的格式：

    #ifndef A_H意思是"if not define a.h"  如果不存在a.h

    接着的语句应该#define A_H  就引入a.h

    最后一句应该写#endif   否则不需要引入

![image](https://user-images.githubusercontent.com/42630862/167447762-a67e1a46-2d6e-4aa0-a86c-00c97fc100c4.png)

## C++ Constructor後面的":"是什麼鬼意思？ (Initialization List 教學)
https://vinesmsuic.github.io/2020/01/09/c++-initializationlists/#initializer-list



