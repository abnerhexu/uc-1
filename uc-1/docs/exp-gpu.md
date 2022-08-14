# 实验：GPU编程

本实验中，我们将体验如何使用GPU完成我们的计算。

### 背景

GPU(Graphics Processing Unit)是一种图形处理器，人们最开始用其进行图像的渲染，主要运用在游戏和视频领域。2010年，我国研究人员首次将GPU应用在高性能计算领域，成功建造出超级计算机天河一号，并在2010年11月发布的TOP500榜单中位列世界第一。

目前应用最广泛的GPU是来自NVIDIA的产品。在NVIDIA GPU上，最底层的是GPU硬件，即各类显卡；操作系统是基于硬件的第一层软件，在操作系统上需要安装GPU的驱动程序CUDA；为了高效进行科学计算，NVIDIA又开发了各类高性能库，例如cuBLAS、cuFFT、cuDNN等；更进一步地，在这之上有Tensorflow、Pytorch等一系列软件和模型。

各种开发软件，以及NVIDIA为我们提供的各类库，使得我们无需关心GPU底层的硬件细节，而能够专注于科学计算应用的编程。有关GPU在硬件方面的内容，我们将会在《计算机体系结构》和《并行程序设计》中进一步介绍。

### Python中的GPU编程

在CUDA中，CPU和主存被称为主机，GPU被称为设备；GPU无法直接读取主存数据，只能读取自身的内存单元（显存）的数据；CPU无法读取显存数据，只能读取主存数据。因此，CPU和GPU必须通过总线进行数据交换。

接下来，我们需要安装CUDA，可以直接访问NVIDIA官网下载驱动并安装；然后是通过pip安装numba。如果环境已经准备好，那么在终端中执行

```bash
nvidia-smi
```

应该会返回如下图所示的结果。

```bash
root@Icb41fbb9c00701bfd:~# nvidia-smi
Mon Jul 25 20:02:10 2022
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.82.01    Driver Version: 470.82.01    CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  On   | 00000000:01:00.0  On |                  N/A |
| 30%   27C    P8    27W / 350W |     10MiB / 24268MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```

然后进入Python交互式编程界面，输入下面的内容，如果检测到GPU就说明一切都配置好了。

```bash
root@Icb41fbb9c00701bfd:~# python
Python 3.8.10 (default, Mar 15 2022, 12:22:08)
[GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from numba import cuda
>>> print(cuda.gpus)
<Managed Device 0>
>>>
```

一般而言，CPU中的程序执行要经过初始化、CPU计算、得到计算结果的步骤，而GPU程序执行一般是初始化并将程序拷贝到显存中，然后CPU调用GPU的多个核心进行计算，之后CPU与GPU异步计算，最后GPU再将得到的结果拷贝会内存。我们来看一个简单的使用GPU运行程序的代码。

```python
from numba import cuda

@cuda.jit
def aPlusb(a, b):
    ans = a + b
    print(ans)

if __name__ == "__main__":
    aPlusb[1, 2](10, 20)
```

我们已经完成了一个最基本的GPU应用！只需要添加装饰器@cuda.jit，就能够将我们的函数搬到GPU上运行。注意，在调用这个函数的时候，我们需要指定GPU以多大的粒度并行进行计算。如上面的代码所示，即用两个线程并行地执行这个函数，实际上则是将这个函数执行了两次。

再次，CPU启动GPU的方式是异步的，也就是说，CPU在启动完GPU核函数之后，就可以接着干别的事情了，例如下面的示例。

```python
from numba import cuda

@cuda.jit
def aPlusb(a, b):
    ans = a + b
    print(ans)

def hello():
    print("hello")

if __name__ == "__main__":
    aPlusb[1, 2](10, 20)
    hello()
```

在这个代码中，很明显我们会看到终端中先打印出了“hello”，然后才接着打印出了2个30。如果希望CPU在GPU执行完毕后再继续执行其他代码，那么我们需要加上cuda.synchronize()，代码变成

```python
from numba import cuda

@cuda.jit
def aPlusb(a, b):
    ans = a + b
    print(ans)

def hello():
    print("hello")

if __name__ == "__main__":
    aPlusb[1, 2](10, 20)
    cuda.synchronize()
    hello()
```

### 实验内容

编写在CUDA上运行的核函数，其功能是实现两个矩阵的乘法运算。

### 实验指导

首先需要完成计算两个矩阵的的乘法计算函数，这应当是非常容易的。

```python
def mul(a, b, c):
    row, ki = a.shape
    column = b[0].shape[0]
    for i in range(row):
        for j in range(column):
            tmp = 0
            for k in range(ki):
                tmp += a[i][k]*b[k][j]
            c[i][j] = tmp

```

为了使其能够在GPU上运行，我们需要将a、b矩阵搬到GPU显存上，并且在显存上直接生成一个结果矩阵c。

```python
from numba import cuda
import numpy as np
R, M, C = 2048, 2048, 2048
a = np.random.rand(R, M)
b = np.random.rand(M, C)
a = cuda.to_device(a)
b = cuda.to_device(b)
c = cuda.device_array((R, C))

```

现在我们要考虑的是在GPU上开多少个线程来完成我们的计算。CUDA将核函数所定义的运算称为线程(Thread)，多个线程组成一个块(Block)，多个块组成网格(Grid)。这样一个Grid可以定义若干个线程，也就相当于若干个并行。在`aPlusb[1, 2](10, 20)`中，我们开了1个Block，每个Block具有2个Thread。

我们的计划是开若干个Block，每个Block包含256个Thread。事实上，CUDA还允许我们开的Block或Thread为二维或三维向量，因此实际上是(16, 16)个Thread。通过内置变量`threadIdx.x`，我们可以知道在某个Block中，本线程为第几个线程；`blockIdx.x`则返回在某个Grid中，是第几个Block；`blockDim.x`表示的是某个Block的大小。那么，某一个线程(x, y)的序号实际上是(x, y\*blockDim.x)。为了并行化，我们需要对`mul`函数做一些修改，将线程号中的x映射为矩阵的行，y映射为矩阵的列。

```python
@cuda.jit
def mul(a, b, c):
    row, column = cuda.grid(2)
    if row < c.shape[0] and column < c.shape[1]:
        tmp = 0
        for k in range(a.shape[1]):
            tmp += a[row, k] * b[k, colum]
        c[row, colum] = tmp
```

至于我们要开多少个Block，与我们的数据规模有关。

```python
blocks_per_grid_x = int(np.ceil(a.shape[0] / 16))
blocks_per_grid_y = int(np.ceil(b.shape[1] / 16))
blocks_per_grid = (blocks_per_grid_x, blocks_per_grid_y)
```

现在我们就可以开始计算了。

计算完成后，需要将结果拷贝回CPU，并在同步后将其打印。

```python
ans = c.copy_to_host()
cuda.synchronize()
print(ans)
```

### 交流与讨论

1\. 解释原因：GPU在人工智能领域有着十分广泛的应用，而CPU则不如CPU出色。

2\. 解释原因：在一台设备上，如果CPU相对落后，而GPU相对高端，运行效果也会大打折扣。

3\. 了解CUDA环境架构，即CPU、GPU、应用程序、CUDA开发库、运行环境、驱动之间的关系。
