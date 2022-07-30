# 实验：Python运行优化

### 背景

一些同学在高中阶段就已经学习过了信息学奥赛。CCF官方允许的编程语言是C和C++（NOI和NOIP已分别于2020年停止使用Pascal）；为什么是这两种语言，而不是Python？这个问题确实不应该由我们来解释（欢迎致电CCF），但有一点是可以确定的，对于OI选手来说，提交的程序在1秒钟之内得到正确结果是最重要的事情之一；而Python实在是太慢了。

我们还是先来观察一下运行速度的差异吧。

```python
x = int(input())
i = 0
ans = 0
while (i < x):
    ans += 1
    i += 1
print(ans)
```

```c
int main(){
    int x, ans = 0, i = 0;
    scanf("%d", &x);
    while (i < x){
        ans++;
        i++;
    }
    printf("%d", ans);
    return 0;
}
```

在我们的测评机上，当输入的n=10000000时，这段Python程序运行完成的时间为1.40秒，而完成同样功能的C程序只需要23毫秒。这个差异是相当显著的。我们当然不能期望我们等待了许久才得到某个答案（况且由于我们编程的失误，它可能还是错误的），因此不论如何我们都应该来探讨一下如何加快Python程序运行的速度。

> 🍈在这一节中，我们不会介绍通用的方法，例如循环展开、减少函数调用、编译器使用更高级别的优化等；我们针对Python程序的特性，介绍Cython和JIT（Just In Time）动态编译技术。

### Cython

从本质上说，Cython就是带有数据类型的Python。在之前的内容中，我们有意无意地忽略了Python变量的数据类型，我们也没有介绍int类型和float类型在内存中存储方式的不同；我们更不会说求大数阶乘时要选取long long unsigned类型，否则会溢出——因为Python就像一个黑盒子，它为我们准备好了这一切。

而准备好这一切的代价，就是Python的执行变得非常缓慢。每一个变量运算前，Python都会不厌其烦地检查这个变量的数据类型。这不禁让我们好奇，如果我们强行指定进行运算的数的数据类型，是不是意味着执行时就可以省下这部分时间呢？Cython为我们提供了非常方便的功能，将我们的程序转换为C代码，从而能够让我们以写Python程序的方式获得更高的效能。

#### 安装

在Windows中，我们需要安装好MinGW-w64，这是一个移植到Windows平台的GCC。如果使用Linux，则通常情况下GCC是已经安装好了的。现在，我们通过`pip install cython`来安装Cython。

#### 使用流程

我们先创建一个hello.pyx文件。文件中写上这样的内容

```python
ans = 0
for i in range(101):
    ans += i
print(ans)
```

事实上，我们是需要来编译这段代码，因此我们还需要额外创建一个setup.py，放在同一文件夹下。

```python
from distutils.core import setup
from Cython.Build import cythonize

setup(
    ext_modules = cythonize("hello.pyx")
)
```

现在，我们可以来编译源文件。只需要在命令行中输入

```bash
python setup.py build_ext --inplace
```

执行完上述命令后，Cython会为你生成hello.pyd文件（Windows）或hello.so（Unix）。需要调用时，只需要打开交互式编程界面，然后输入import hello，我们就能调用我们刚刚写过的代码，并且输出正确答案。

```python
Python 3.6.9 (default, Mar 15 2022, 13:55:28)
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import hello
5050

```

很不错！它输出了正确答案。但是从整个流程来看，似乎写Python代码、编译、引入、运行，这套流程显得略有些复杂。所以我们希望有一个更方便的办法。

#### Cython的简单编译

现在，我们能像写“纯粹”的Python代码一样来工作了。在上一个实验中，我们曾经给出过一个时间复杂度为$O(n^2)$的计算斐波那契数列的函数实现。现在，我们将函数原封不动地复制到fib.py中，然后打开交互式编程界面，输入

```python
import pyximport; pyximport.install()
import fib
print(fib.fib(100))
```

好了，现在我们就回到了之前的步骤，而不需要每次手动去编译了。

pyximport是Cpython中包含的可以直接加载.pyx文件的模块。由于版本的更新，现在加载文件时默认是使用Python3，而只有当文件为.pyx时才以Python2的方式加载。为了让我们的代码保持良好的一致性，我们将计算斐波那契的函数写进了.py文件而不是.pyx文件中，这样就是以Python3的方式进行加载的。

> 🍈选择合适的方式对程序进行评估：使用Cython编译后，程序的运行速度应当是有一定的提升的。如何更准确地测量程序运行时间，并减小总体误差？

此外，我们也可以使用Cython调用C的库。至于如何操作，请RTFM。

### 开源JIT编译器：Numba

我们先来介绍一下，一段Python代码是如何在计算机上运行起来的。一个标准的回答是：运行Python脚本时，Python的解释器首先创建一个字节码编译器，将源代码转换成字节码，而字节码只能在虚拟机上运行，因此还要再通过Python虚拟机才能与硬件设备进行交互。

JIT则在程序运行时直接将程序编译为机器语言，能够直接被硬件接受并执行。因此，速度会快很多；事实上，这与编译好的C语言并无太大的差别。

有一些Python语言的特性是支持的，例如while循环、for循环、if语句等等，但是Numba目前还不支持某些语句。这一点可以通过RTFM获得更详细的说明。

#### 安装

Numba的安装方式我们应该是再熟悉不过了，打开命令行，输入

```bash
pip install numba
```

略微等待就能完成安装工作。

#### 快速上手

事实上我们也只是在正常地写Python代码，和我们之前的习惯并无二致。我们来写一个计算两个矩阵相加的程序片段，并通过测试来观察二者所需的时间差异。

```python
import numba
import numpy as np

row, column = 1200, 2400
x = np.random.rand(row, column)

@numba.jit()
def jit_sum(a):   
    ans = 0
    for i in range(row):   
        for j in range(column):
            ans += a[i][j]   
    return ans

print(jit_sum(x))
```

```python
import numpy as np

row, column = 1200, 2400
x = np.random.rand(row, column)

def simple_sum(a):   
    ans = 0
    for i in range(row):   
        for j in range(column):
            ans += a[i][j]   
    return ans

print(simple_sum(x))
```

在jupyter中，我们可以在每个单元格的第一排添加`%%time`的魔术代码，对该单元格的运行时间进行监测。在我们的测评机上，使用了JIT编译的版本要比没有使用JIT的版本快上5倍，而从我们的代码本身来看，我们只添加了两句额外的语句：引入numba库、添加一个numba的**装饰器**。装饰器的本质是Python中的某个函数或者类，其功能是在我们不需要修改原函数的情况下，能够帮助原函数拥有某些功能。

#### 编译的开销

如果我们将上面的代码中row和column分别改为较小的数，例如62和100，我们很容易发现，不使用JIT编译的代码运行起来要比使用JIT编译的代码快得多。这说明编译是存在开销的，而当数据规模很小时，编译带来的开销不可忽视。

```python
import numba
import numpy as np
import time

row, column = 1200, 2400
x = np.random.rand(row, column)

@numba.jit()
def jit_sum(a):   
    ans = 0
    for i in range(row):   
        for j in range(column):
            ans += a[i][j]   
    return ans

time0 = time.time()
print(jit_sum(x))
time1 = time.time()
print("%.4f", time1 - time0) # 编译和一次函数调用的近似时间

time0 = time.time()
print(jit_sum(x))
time1 = time.time()
print("%.4f", time1 - time0) # 一次函数调用的近似时间
```

如果我们进行两次函数调用，我们会发现第二次函数调用的时间显著降低。这就对了，通过一次的编译，我们能为之后的函数调用提供更优的性能。

### 实验内容

编写一个方阵乘法函数，函数原型为

```python
def mul(x, y, length)
```

其中x和y是两个不同的方阵，数据类型为numpy.ndarray；length是一个整数，表示方阵的大小，对于一个$n\times n$的方阵，`length = n`；函数的返回值为一个方阵。

*   实现函数mul；

*   尽可能准确地估计函数的编译时间；

*   记录length = $1.0\times10^2$，$1.5 \times 10^2$，$2.0×10^2$，$2.5×10^2$时，不使用JIT编译的代码与使用JIT编译的代码在运行时间上的差异；

*   用Cython实现相同的函数，比较在上述数据规模下的时间差异；

*   (\*选做)编写相同功能的C语言或Fortran语言程序，比较时间差异；

*   绘制曲线描述上述差异。

#### 实验提示

> 🍈**拒绝内卷：** 完成选做项目不会有额外分数。

### 交流与讨论

1\. 随着length的线性增大，运行的时间是线性增加的吗？如果不是，是哪些因素影响了运行时间的变化？

2\. 什么样的函数在使用了JIT编译后速度会有显著提升，而什么样的函数不一定是这样？

3\. 装饰器@vectorize还可以将一个函数向量化。了解向量化的原理，并尝试使用向量化方法加速函数。
