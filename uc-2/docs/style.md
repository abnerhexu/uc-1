# 代码风格建议

写出可移植的、高可用的、容易复用的代码是我们追求的目标。在大型应用中，常常有很多人一起参与同一个项目的开发；对于像Linux内核这样的大型开源软件，还需要集合全世界的开源力量，我们很容易在这些大型软件上看到不同的代码风格，不论采取什么样的风格，至少我们需要保证代码的易读性，因此我们推荐一些常用的代码风格，供大家参考。

#### 规则1

一行代码只做一件事情，即一行代码只包含一个分号。

推荐的风格

```c++
int a, b, c;

```

不推荐的风格

```c++
int a; int b; int c;
```

#### 规则2

`{ }`之内的代码有相同的缩进；嵌套的`{ }`照此处理。

推荐的风格

```c++
int main(){
    printf("%d", 42);
    return 0;
}
```

不推荐的风格

```c++
int main(){
    printf("%d", 42);
  return 0;
}
```

#### 规则3

赋值操作符、比较操作符、算术操作符、逻辑操作符、位域操作符，如“=”、“+=” “>=”、“<=”、“+”、“ \*”、“%”、“&&”、“||”、“<<”,“^”等二元操作符的前后建议**适当**加空格。

推荐的风格

```c++
int main(){
    std::cout << a + b << std::endl;
    return 0;
}

```

不推荐的风格

```c++
int main(){
    std::cout<<a+b<<std::endl;
    return 0;
}
```

#### 规则4

逗号和不代表一行结束的分号与后续内容之间保留一个空格。

推荐的风格

```c++
void foo(int x, int y){
    for (int i = x; i <= y; i++){
        cout << i << endl;
    }
}
```

不推荐的风格

```c++
void foo(int x,int y){
    for (int i=x;i<=y;i++){
        cout<<i<<endl;
    }
}
```

#### 规则5

在每个类声明之后、每个函数定义结束之后都要加空行。

推荐的风格

```c++
struct node{
    int data;
    node * next;
};

int main(){
    node node0;
    cout << node0.data << endl;
    return 0;
}
```

不推荐的风格

```c++
struct node{
    int data;
    node * next;
};
int main(){
    node node0;
    cout << node0.data << endl;
    return 0;
}
```

#### 规则6

整个程序中不使用中文拼音，而采用英文单词或其组合。建议使用有意义的英文单词作为变量名，并且采用驼峰命名法或下划线命名法。

```c++
bool game_over_flag = 1; //下划线命名法
bool gameOverFlag = 1; // 小驼峰命名法
bool GameOverFlag = 1; // 大驼峰命名法
```
