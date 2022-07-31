# 面向对象编程

从这一章节开始，我们要学习关于面向对象编程的基本概念和方法，并且这些概念和方法在其他的语言上是相通的，例如C++和Java。说到语言，一些语言是面向过程的，例如C语言；而一些语言是完全面向对象的，例如Java；还有一些语言支持这两种模式，例如Python和C++。

说到这两种编程范式，面向过程（Procedure Oriented, PO）我们应该还是比较熟悉的。面向过程指的就是我们在通过编程解决问题时，将问题拆分成更小的问题，然后针对这些问题设计函数。按照一定的顺序把这些函数执行完毕，问题就得到解决——我们几乎一直都在显式地使用这样的编程模式。

而面向对象（Object Oriented, OO）是在解决问题的时候，将我们面对的事物抽象成为对象的概念，这些对象具有一定的自身的属性和方法，通过这些对象去执行这些方法来使问题得到解决——事实上我们也一直都在使用面向对象的方法，例如我们之前创建的列表、数组、字符串，它们都是Python中的对象；而它们具有的`append`、`shape`、`split`这些操作叫方法。

### 5.1 面向对象编程的基本概念

面向对象有三大基本特性——封装、继承和多态。

封装是指把客观事物抽象成为类，而类具有一定的属性和方法，可以设置只有某些特定的类或者对象能够访问这些数据或调用这些方法。在一个对象的内部，某些属性和方法可能是私有的，就防止了未经许可的访问。因此，对象内部的数据可以得到不同程度的保护。

继承是指让某个对象具有另外某个对象的属性和方法。例如，假设我们有一个名叫“哺乳动物”的类，那么我们可以定义一个名叫“大熊猫”的类，作为“哺乳动物”类的一个子类。子类可以继承父类的所有功能，并且可以针对性地做改动。

多态是指同一个类的同一个方法在不同的情形下具有不同的形式，它使得对象内部不同的数据结构能够共享相同的接口。也就是说，可能对不同的数据类型的具体操作是不同的，但是这些操作都被抽象成为统一的接口，只需要做对应接口的调用就可以了。

对于单片机和嵌入式设备等对性能敏感的场景，一般采用面向过程的开发模式；而面向对象模式虽然对性能要求略高（因为初始化每一个实例都有性能开销），但是能够帮助我们设计出低耦合的系统，使软件系统更容易维护。

### 5.2 Python中的面向对象

#### 5.2.1 类和对象的创建与访问

在Python中，创建类是很容易的，只需要使用关键字`class`。

```python
class Foo:
    def hello(self, name):
        print("hello,", name)

```

现在我们可以创建一个Foo类的实例。

```python
obj_Foo = Foo()
```

然后可以调用obj\_Foo中的hello方法，传入参数name，就会打印出相应的内容。

```python
obj_Foo.hello("Panda")
```

有几个点需要注意，一是类名按照习惯使用大驼峰命名法；二是类中定义的函数叫做类的方法，它们的第一参数必须是self；三是如果类中有数据，还需要定义类的构造函数`__init__`，例如

```python
class Foo:
    def __init__(self, identity):
        self.identity = identity
    def hello(self, name):
        print("hello,", name)
    def checkid(self):
        print("id:", self.identity)
```

这里，我们创建实例的时候就必须先传入identity的值，例如，

```python
obj_Foo = Foo(210100110487)
obj_Foo.checkid()
```

关于此处self.identity的含义，可以简单的理解为是将传入的某个参数identity（对象中的方法无法访问）先赋值给self.identity，这样对象中的方法才可以访问和使用。事实上，self代表的是我们创建的类的具体实例，它指向我们创建的实例的内存地址。

此外，我们也可以在定义类的时候就添加某些变量（类的属性）。以两根下划线开头的变量是私有变量，这意味着不能通过外界访问。

```python
class Foo:
    gender = "male"
    __grade = 100
    def __init__(self, identity):
        self.identity = identity
    def hello(self, name):
        print("hello,", name)
    def checkid(self):
        print("id:", self.identity)
    def checkg(self):
        print(self.__grade)
```

这个时候，我们可以访问`identity`，也可以访问`gender`，但不能访问`__grade`。通过对象内部的方法才可以访问到`__grade`的值；注意，对象内访问对象的数据，都需要通过`self.var`的形式。

```python
obj_Foo = Foo(210100110487)
print(obj_Foo.gender)
print(obj_Foo.identity)
obj_Foo.checkg()
print(obj_Foo.__grade) # 会报错
```

同样，我们也可以定义私有的方法。

```python
class Foo:
    gender = "male"
    __grade = 100
    def __init__(self, identity):
        self.identity = identity
    def __checkg(self):
        return self.__grade
    def hello(self):
        print("Grade:", self.__checkg())
```

然后只能通过类中的其他方法来调用。

```python
obj_Foo = Foo(210100110487)
obj_Foo.hello()

```

关于变量和方法名称中下划线的使用：不加下划线表示是一个public类型的属性或方法，任何对象都可以调用；加一个下划线表示是protected类型，只能通过其本身和子类进行调用；两个下划线表示是private类型，只能由这个类本身访问。

#### 5.2.2 类的继承

现在我们有一个叫`University`的类。

```python
class University:
    def __init__(self, name, identity):
        self.name = name
        self.identity = identity
    def applyfor(self):
        print(self.identity)
```

我们可以构造一个叫`NTTC`的类，并继承于类`University`。这个子类也可以有自己的构造函数，以及添加进自己特有的属性和方法。添加了自身的构造函数后，可以通过super()来调用父类的构造函数。弗雷德方法也是可以重写的，例如，

```python
class NTTC(University):
    def __init__(self, name, identity, Aplus):
        super().__init__(name, identity)
        self.Aplus = Aplus
    def achievements(self):
        print("Milkyway")
    def applyfor(self):
        print("id:", self.identity)
```

然后可以调用父类的函数。

```python
nudt = NTTC("NUDT", 9102, "CS")
nudt.applyfor()
```

当然，也可以继承多个类，只需要在子类定义时写入全部父类的名称即可。

#### 5.2.3 运算符重载

我们只需要举一个简单的例子，相信聪明的你一定能够明白其余类似的操作。

> 🍈通过类定义新的数据类型：复数，并使其支持加法的运算。

首先我们需要定义一个类。

```python
class Complex:
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag
```

我们很容易想到的是，在这个类里面定义某种方法，例如加。

```python
def add(self, b): # 其中b是另一个复数类型变量
    ans = Complex(0, 0)
    ans.real = self.real + b.real
    ans.imag = self.imag + b.imag
    return ans
```

所以我们可以通过`c = a.add(b)`的方式将`a`和`b`相加，并把值赋给`c`。

```python
a = Complex(1, 2)
b = Complex(1, 3)
c = a.add(b)
```

现在，我们希望我们的代码能够更自然一些。就像两个实数相加可以使用运算符`+`一样，我们希望我们定义的这个数据类型也可以这样——这就要用到运算符重载。Python提供了运算符重载的特殊函数，我们可以定义它们。

```python
class Complex:
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag
    def __add__(self, other):
        r = self.real + other.real
        i = self.imag + other.imag
        return Complex(r, i)
a = Complex(1, 2)
b = Complex(1, 3)
c = a + b
print(c)
```

但是这引出了另一个问题，我们希望`c`也能自然地打印出来，而不是像现在一样返回的是一串内存地址。

```python
<__main__.Complex object at 0x0000019DD3D99CC8>
```

这里，我们要用到第二个特别的方法，`__str__`。

```python
class Complex:
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag
    def __add__(self, other):
        r = self.real + other.real
        i = self.imag + other.imag
        return Complex(r, i)
    def __str__(self):
        return "{}+{}i".format(self.real, self.imag)
a = Complex(1, 2)
b = Complex(1, 3)
c = a + b
print(c)
```

好了，打印出了结果$2+5\mathrm i$。现在请聪明的你来解决接下来的问题。

类似的函数如下图所示，请选用合适的函数，完成两个复数的减法、乘法的运算。

| 运算符  | 表达式    | 重载函数           |
| ---- | ------ | -------------- |
| +    | x+y    | `__add__`      |
| -    | x-y    | `__sub__`      |
| \*   | x\*y   | `__mul__`      |
| \*\* | x\*\*y | `__pow__`      |
| /    | x/y    | `__truediv__`  |
| //   | x//y   | `__floordiv__` |
| %    | x % y  | `__mod__`      |

类似的，还有比较运算符的重载。

| 运算符 | 表达式   | 重载函数     |
| --- | ----- | -------- |
| <   | x < y | `__lt__` |
| <=  | x<=y  | `__le__` |
| ==  | x==y  | `__eq__` |
| !=  | x!=y  | `__ne__` |
| >=  | x>=y  | `__ge__` |
| >   | x>y   | `__gt__` |
