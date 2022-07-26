# 从Python到C++

!!! note "阅读提示"

    **这是第一周的课外阅读材料。不阅读它也没有关系，不会影响主线学习进程。**

### 关于两门课程

在《大学计算机基础》中，你已经了解了最基本的关于计算机科学的部分内容，例如你了解了信息在计算机中是如何编码和表示的，在实现“TOY计算机”的过程中你从程序员视角观察了计算机的硬件和软件系统，你还简单了解了计算机网络的相关概念和数据库的使用，在上面的过程中，Python作为一门编程语言为你的学习带来了很大的方(tong)便(ku)，而这学期我们将通过《C++程序设计》来对这一门编程语言有进一步的了解，并且使用它完成一系列编程任务。

在《大学计算机基础》中，我们关心的是 **概念** ，例如抽象、递归、状态机，而《C++程序设计》这门课上我们还要将视线深入到 **代码** ，通过对问题的建模，写出实用的、优雅的、可移植的代码（这有一点困难，我们的课程不会涉及太多）。

### 关于两门语言

#### 应用

Python是一门很热门的语言(~~相信大家都已经在微信朋友圈看过了无数Python的广告~~)，在人工智能和计算科学领域非常火热；而C++在数据科学领域也有很广泛的应用，例如著名的机器学习库Pytorch和Tensorflow尽管提供了Python的API以便我们方便地调用，但是其后端依然是C++实现的。

C++也更适合用于开发大型的软件，例如Chrome和Firefox这两个著名的浏览器，就主要是基于C++开发的；Windows等操作系统、Office办公软件等也是使用C++开发的；就连控制勇气号在火星上探索的程序中，也有C++的踪影。

#### 学习C++的原因

首先，作为计算机相关专业的学生，学习C++是十分必要和有用的。CUDA编程、计算机网络计算机视觉等课程会广泛使用这一语言；信息学竞赛(OI)中，多数选手也会使用C++作为编程语言。

其次，相较于Python，C++程序运行更快。一方面是与两种语言的类型和代码执行的流程不同有关，另一方面也与Python解释器需要不断检查数据类型有关。如果你使用OJ测评具有相同时间和空间复杂度的Python和C++代码，你会发现C++代码的内存占用会更少、运行时间会更短。

还有一点，C++是更低级(low level)的语言(注意！C++依然是一门高级语言，只是说，相对于Python而言，更接近硬件一些)，这意味着我们能够控制的事情更多。例如，Python中没有指针和指针变量的概念，而指针，正是C语言(也包括C++)的灵魂。在学习时，我们还应当关注在《大学计算机基础》中曾经介绍过的内容，例如二进制原码、反码、补码，内存地址等，这些内容将有助于你理解C++程序设计中的某些概念。

### C++程序

让我们来剖析一个最简单的C++程序。就像把大象装进冰箱的问题一样，要写好一个C++程序，我们首先需要关心基本步骤。

```c++
#include <iostream>
int main(){
    std::cout << "Hello, world!" << std::endl;
    return 0;
}

```

第一步，我们将iostream(Input and Output Streams, I/O Streams)这个标准库引入进来，就像大家熟知的在Python中我们通过`import numpy`来引入NumPy包一样。非常容易。

第二步，我们定义了一个函数，叫main()函数。在大多数可以单独执行的C++程序中，程序都是从这个函数开始执行的[注1]。

第三步，我们在main()函数中写上std::cout，代表我们调用这个标准库中的cout(读作C out)，表明我们开始输出某些内容，符号`<<`表示输出，然后双引号引出字符串，然后通过std::endl打印换行符。在C++中，每一条语句结束以后，需要使用英文状态下输入的分号来表示这一条语句的结束(引入头文件的时候不需要)。

第四步，我们定义的main()函数前有一个int，代表我们的这个函数要返回一个整数。因此我们在最后return 0即可(通常返回非零代表程序异常退出。在线上测评系统(Online Judge, OJ)中，一般只接受return 0这种写法)。

这就是最简单的C++程序。我们为什么要写`std::cout`，是因为我们在Python中已经见过类似的问题，就是我们定义的函数可能与引入的库的函数重名。在Python中，我们可以通过`import numpy as np`这种别名的方式来解决，在C++中，我们是通过标明其是标准库的类来实现。

如果实在觉得难受，我们可以增加一条语句，这样我们只需要写`cout`和`endl`，而不需要前面的`std::`了。改写后的代码如下所示。在小型程序中这可能很方便，但对于大型程序，我们还是建议使用上面的写法。

```c++
#include <iostream>
using namespace std; // 这是我们添加的一条语句
int main(){
    cout << "Hello, world!" << endl;
    return 0;
}
```

### C++的历史

这也是一段C++代码。

```c++
#include "stdio.h"
#include "stdlib.h"

int main(int argc, char *argv) {
    asm("sub $0x20,%rsp\n\t"
        "movabs $0x77202c6f6c6c6548,%rax\n\t"
        "mov %rax,(%rsp)\n\t"
        "movl $0x646c726f, 0x8(%rsp)\n\t"
        "movw $0x21, 0xc(%rsp)\n\t"
        "movb $0x0,0xd(%rsp)\n\t"
        "leaq (%rsp),%rax\n\t"
        "mov %rax,%rdi\n\t"
        "call __Z6myputsPc\n\t"
        "add $0x20, %rsp\n\t"
        );
     return EXIT_SUCCESS;
}
```

事实上，应该说这是一段伪装成C++的汇编语言代码。汇编语言的优点在于，我们仅仅使用最简单的指令就能完成全部代码的编写(回想一下Python中各种包的各种用法，是不是觉得很头大，所以《大学计算》开卷考试也不是没有道理)，并且汇编语言 的执行比C++更快，最重要的是，我们能够控制每一步我们的程序都做了些什么操作，我们能完全地控制我们的程序。

那为什么不直接使用汇编语言呢？答案应当也是很显然的，为了完成简单的任务，我们要写大量的语句，并且这些汇编语言非常难以理解；最后，在不同体系结构的机器上，我们的汇编语言通常可移植性非常差。

为了解决这个问题，人们发明了C语言，也发明了编译器。我们只需要编写容易被我们理解的C语言代码，编译器就能帮我们自动地翻译为晦涩难懂的汇编语言。1972年，贝尔实验室的Dennis M.Ritchie发明了C语言，C语言非常成功，因为它非常简单；但同时因为它太简单了，没有对象(object)和类(class)，我们在使用C语言开发大型项目时就非常困难。

1983年，Bjarne发明了C++，C++完全兼容C语言的语法和特性，并且C++引入了许多高级的特性，例如我们刚刚见到过的`<<`来帮助我们输出某些内容。从诞生到现在，C++的标准也在不断进步，目前最新的标准是C++20；考虑到历史的原因，在我们本课程的学习中，我们主要使用C89标准编写C语言代码，仅仅在数据读写中使用C++提供的面向对象方法。

### 从Python迁移到C++

#### 数据类型

C++是一个强调数据类型的语言。在Python中，我们想定义一个变量太容易了，只需要写`a = 1`就能把1这个整数赋给变量`a`。

在C++中，变量使用和赋值前需要先声明，即标示它的类型。例如，当我们要创建一个名为`a`的`int`类型的变量，我们需要这样写：

```c++
int a;
a = 1;
```

常见的类型如下表所示(以x86-64机器，gcc 9.3.0为例)。

| 类型           | 数据范围                                   | 示例                 |
| ------------ | -------------------------------------- | ------------------ |
| int          | -2147483648 ~2147483647                | int a = -1         |
| unsigned int | 0~4294967295                           | unsigned int a = 1 |
| long long    | -9223372036854775808~9223372036854775807 | long long a = 100  |
| char         | -128~127                               | char a = 'A'       |
| float        | 暂不讨论                                   | float a = 1.0      |
| double       | 暂不讨论                                   | double a = 2.0     |

此外，一旦定义了某个变量的类型，我们就不能更改它的类型。例如，我们在定义了int型变量a后就不能再将一个字符串赋值给它。

```c++
// 下列写法是错误的
int a = 1;
a = "abc";
```

[注1]: 以下内容不需要初学者深究，在之后的《计算机系统》、《操作系统》等课程中还会继续介绍。事实上main()函数的开始和结束并不代表着整个程序的开始和结束。要想知道一个程序究竟是从哪里开始运行的，我们可以通过gdb调试程序，事实上，上面这段打印出Hello, world!的代码在main()开始执行以前，首先[ld-linux-x86-64.so](http://ld-linux-x86-64.so "ld-linux-x86-64.so") 加载了 libc。作为初学者，这些都不是我们该关心的内容。
