---
title: "Hello, World"
date: 2020-08-29T17:43:10+08:00
draft: false
---

# Hello, World

对于程序员来说，Hello, World 一定是不陌生的了。每次学一种新的编程语言的时候，第一个代码示例都是hello world。

### Hello World的由来

> 1972年，贝尔实验室成员布莱恩·柯林汉撰写的内部技术文件《A Tutorial Introduction to the Language B》首次提到了Hello World这字符串。当时，他使用B语言撰写了第一个使用参数的Hello World相关程序。
>
> 1972年，布莱恩·柯林汉和丹尼斯·里奇基于B语言写成C语言后，在他们撰写的《C程序设计语言》使用更简单的方式展示Hello World。
>
> 自此，Hello World成为了电脑程序员学习新的编程语言的传统。

在github上，有一个repo就记录了各种语言的Hello world的写法，大家可以自己去看一下，很有意思[2]。

### C++的Hello World 

C++的 hello world ： 

```cpp
#include <iostream>
int main(){
  std::cout<<"Hello, World"<<std::endl;
  return 0;
}
```

我们来分析一下这简短的5行代码包含了哪些内容

* `#include <iostream>`

  - 程序的编写一直在强调模块化，这句话表明了C++引用其他模块的方式，就是使用include关键字，后边跟着你要引用的类或对象所在的文件。
  - `iostream`是输入输出流功能的函数或者类所在的文件名，文件名用`<>`包裹。

* `int main(){` 

  - `int` 表明了函数的返回类型；不同python这种脚本语言，C++要求要有一个代码运行的入口点，C++标准将这个入口点函数称之为`main`。

* `std::cout<<"Hello, World"<<std::endl;`

  - `std`是一个命名空间，C++使用命名空间来避免命名冲突；std是standard的缩写，每一个编译器都需要实现std命名空间内的内容。

  - `"Hello, World"`被**引号**包裹，表明C++的字符串使用`""`来区分的。

  - `endl`这是C++所定义的换行符。其定义为

    ```cpp
    template< class CharT, class Traits >
    std::basic_ostream<CharT, Traits>& endl( std::basic_ostream<CharT, Traits>& os );
    ```

  - `;`C++语句通过分号进行分割

* `return 0;`该函数的返回值为`0`。

在稍后的章节里，我们会讨论跟上面有关的内容，包括：

- C++ include 导致的性能问题
- 返回值作为一种错误检验机制，跟异常的对比
- 输入输出流是怎么回事
- main函数是怎么被调用的
- std 标准模板库的介绍



## 参考

1. ["Hello, World!" program](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program0)
2. [Aniket965/Hello-world](https://github.com/Aniket965/Hello-world)
3. [std::endl](https://en.cppreference.com/w/cpp/io/manip/endl)

