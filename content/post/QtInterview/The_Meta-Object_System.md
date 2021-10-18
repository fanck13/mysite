---
title: "Qt 元对象系统"
date: 2020-08-29T17:43:10+08:00
draft: false
---

# Qt 元对象系统

[The Meta-Object System | Qt Core 5.15.6](https://doc.qt.io/qt-5/metaobjects.html#:~:text=The Meta-Object System Qt's meta-object system provides the,that can take advantage of the meta-object system.)

Qt 元对象系统提供了3个功能：

* 用于对象之间通讯的信号槽机制
* 运行时类型信息
* 动态属性系统

元对象系统基于以下元素构建：

* `QObject` 作为所有利用元对象系统的类的基类；
* 位于 `private`中的宏 `Q_OBJECT`，是用来使元对象系统生效的，例如动态属性，信号和槽；
* 元对象编译器可以给所有继承自`QObject`的子类添加用于实现元对象系统的必要的代码。

moc 会读取C++ 源文件，如果其发现在一个或多个类的声明中包含有宏`Q_OBJECT`，会产生另外一个C++ 源文件，这个新产生的源文件包含这些类的元对象代码，这个新文件或者会被`#include`到类的元文件中，或者会被编译到或者链接到类的实现中。

为了提供对象之间的通讯机制-信号和槽（引入这个系统的主要原因），元对象代码还提供了额外的特性：

* `QObject::metaObject()`会返回类的相关联的元对象；
* `QMetaObject::className()`在运行时以字符串的形式返回类的名字，可以不需要C++ 编译器所支持的原生的运行时类型信息（RTTI）；
* `QObject::inherits()`函数返回一个对象是否是继承自特定的类的实例，两者都应在`QObject`的继承树中；
* `QObject::tr()`和`QObject::trUtf8()`会翻译字符串，用于国际化；
* `QObject::setProperty()` and `QObject::property()`会通过名字动态的获取和设置属性；
* `QMetaObject::newInstance()` 创建类的一个新实例。

通过`qobject_cast()`可以对于`QObject`及其子类进行动态的转换。`qobject_cast()`跟`dynamic_cast()`的作用类似，其优点是不需要运行时类型信息，并且可以跨动态库工作。它尝试将其参数转换到指定的类型，如果成功的话，返回一个指向正确对象的非0指针（运行时确定），失败的话返回`nullptr`。

例如，假设`MyWidget`继承自`QWidget`，并且声明了`Q_OBJECT`宏。

```c++
QObject *obj = new MyWidget;
```

变量`obj`，其类型为`QObject*`，实际上指向的是`MyWidget`对象，所以能够正确的转化：

```c++
QWidget *widget = qobject_cast<QWidget *>(obj);
```

强烈建议在所有`QObject`的子类中添加`Q_OBJECT`宏，无论其是否真的使用信号，槽和属性。