---
title: "C++ 面试题"
date: 2020-08-29T17:43:10+08:00
draft: true
---

# C++ 面试题

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