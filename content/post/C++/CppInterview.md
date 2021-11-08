---
title: "C++ 面试题"
date: 2020-08-29T17:43:10+08:00
draft: false
---

### 1. 在有继承关系的父子类中，构建和析构一个子类对象时，父子构造函数和析构函数的执行顺序分别是怎样的？

```c++
class Base1
{
public:
    Base1(){std::cout<<"Cons Base1"<<std::endl;}
    ~Base1(){std::cout<<"Dest Base1"<<std::endl;}
};

class Base2
{
public:
    Base2(){std::cout<<"Cons Base2"<<std::endl;}
    ~Base2(){std::cout<<"Dest Base2"<<std::endl;}
};

class Derived:public Base2, Base1
{
public:
    Derived(){std::cout<<"Cons Derived"<<std::endl;}
    ~Derived(){std::cout<<"Dest Derived"<<std::endl;}
};

int main()
{
    Derived d;
    return 0;
}
```

运行结果：

```bash
Cons Base2
Cons Base1
Cons Derived
Dest Derived
Dest Base1
Dest Base2
```

### 2. 在有继承关系的类体系中，父类的构造函数和析构函数一定要申明为 `virtual` 吗？如果不申明为 `virtual` 会怎样？

```

```

### 3. 为什么构造函数不能是`virtual`?

As Bjarne himself explains in his [C++ Style and Technique FAQ](https://www.stroustrup.com/bs_faq2.html#virtual-ctor):

> A virtual call is a mechanism to get work done given partial information. In particular, "virtual" allows us to call a function knowing only an interfaces and not the exact type of the object. To create an object you need complete information. In particular, you need to know the exact type of what you want to create. Consequently, a "call to a constructor" cannot be virtual.

### 4. 详细介绍下`main`函数

* `main`函数的原型为 `int main(int argc, char* argv[], char* envp[])`。`argv`和`argc`为命令行参数，保存程序运行时的命令行的参数，`argv`表示参数的数量，`argv`是一个`char`指针数组，每个`char`指针都指向实际的参数内容。`envp`表示程序运行时的环境变量，`main`函数并没有指定环境变量的数量，使用时，通过`envp[n] == nullptr`来判断该数组的结束。

  ```c++
  #include <iostream>
  
  int main(int argc, char **argv, char **envp)
  {
      for(int i = 0; i< argc; i++)
      {
          std::cout<<argv[i]<<std::endl;
      }
  
      int j = 0; 
      while(envp[j] != nullptr)
      {
          std::cout<<envp[j]<<std::endl;
          j++;
      }
      return 0;
  }
  ```

  输出结果：
  ```bash
  ./output.s
  LD_LIBRARY_PATH=/opt/compiler-explorer/gcc-11.2.0/lib:/opt/compiler-explorer/gcc-11.2.0/lib32:/opt/compiler-explorer/gcc-11.2.0/lib64
  ASAN_OPTIONS=color=always
  UBSAN_OPTIONS=color=always
  MSAN_OPTIONS=color=always
  LSAN_OPTIONS=color=always
  HOME=/app
  ```
  
* `main`的返回类型为`int`，但是可以省略`return`。即下面的例子也是可以通过编译的

  ```c++
  int main()
  {
  }
  ```

  其返回类型不能通过`auto`进行推导，也就是说`auto main()`是错误的，即便你在函数体里面写了`return 0;`。

  下面代码无法编译

  ```C++
  auto main()
  {
      return 0;
  }
  ```

  下面代码可以编译

  ```c++
  auto main()->int
  {
      return 0; //This line is not necessary
  }
  ```

  * `main`函数不能进行递归调用。
  * 
### 5.模板

![](..\..\image\cpp_function_template.png)



![](..\..\image\cpp_class_template.png)

1. 模板定义了如何生成一系列函数，这些函数有相似的行为。
2. `extern template` 
3. 特化和函数重载之间存在冲突

  
