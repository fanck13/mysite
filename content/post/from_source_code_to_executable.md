---
title: "From_source_code_to_executable"
date: 2020-08-30T13:47:43+08:00
draft: true
---

# 从源代码到可执行文件

从源代码到可执行文件要分为三个阶段，分别是预处理，编译和链接。我们还是以helloworld代码代码为示例，简单的学习一下这三个阶段。

## 预处理

我们将helloworld的代码保存到hello-world.cpp中，然后执行以下命令，就能知道hello-world.cpp文件的大小了。

```sh
$ wc -l hello-world.cpp
  6 hello-world.cpp
```

然后，我们通过g++来获取预处理后的代码，在查看下预处理后的代码的行数.

```sh
$ g++ -E hello-world.cpp -o hello-world.pre
$ wc -l hello-world.pre
  40753 hello-world.pre
```

我们可以看到，经由预处理，一个简单的源文件会膨胀成一个很大的文件。那么，预处理究竟处理了哪些内容呢：

* include 会**递归**的将所有的涉及到的头文件都复制到源文件中。在上面的例子中，先将iostream中的内容复制到hello-world.cpp中来；iostream又include了streambuf istream ostream等文件，所以这些文件也会被复制到hello-world.cpp中来，然后就这么一层层的做下去。这其实是使源文件膨胀的主要原因。
* #define，所有的宏定义也会这这一阶段展开，也会造成源文件的膨胀。
* undef, if, ifdef,ifndef,else, elif, endif 等会将不满足的代码删除掉，这样会减少源文件的大小
* line error 这些可以给开发者一些编译器的消息

其实上面描述的也就是include一直被大家所诟病的地方，编译是以源文件为单元进行的，也就是一个cpp文件就是一个编译单元，而它们之间对于引用的头文件都是独立的。举个例子：

有两个头文件a.cpp 和 b.cpp，它们都include了iostream，那么这两个文件都会膨胀到这么大，这其实是一种浪费。C++20 通过引入 import 来解决这个问题。

**Note:** 实际上，预处理只是生成`hello-world.pre`其中一个的一个阶段。整个的阶段是翻译阶段，而预处理只在整个翻译阶段九个phase的第4个phase。

## 编译



## 链接





## 参考

1. [Translation Phases](https://zh.cppreference.com/w/cpp/language/translation_phases)
2. [Preprocessor](https://en.cppreference.com/w/cpp/preprocessor)