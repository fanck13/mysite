---
title: "Qt 对象树和所有权"
date: 2020-08-29T17:43:10+08:00
draft: false
---

### 概览

`QObject`及其子类的对象以对象树的形式组织。当创建一个`QObject`对象，并以其他对象作为父亲节点，父亲节点会将其加入到孩子列表中，当父亲节点被删除时，该对象也会被删除。这种方法非常适合Gui对象。例如一个`QShortcut`（键盘快捷键）对象是对象的窗口的孩子，当用户关闭窗口时，这个快捷键也被删掉了。

`QQuickItem`，Qt Quick模块的基础的可视化模块，继承自`QObject`，有一个可视化父节点的概念，不同于`QObject`的父亲节点。一个对象的可视化父亲节点不必跟其父亲节点相同。请参考 [Concepts - Visual Parent in Qt Quick](https://doc.qt.io/qt-5/qtquick-visualcanvas-visualparent.html) 。

`QWidget`是Qt Widgets模块的基础类，扩展了父亲-孩子关系。一个孩子节点变成了一个子控件，它在父控件的坐标系统内显示，并被其父控件的边界所截断。例如，当一个应用程序删除消息对话框的时候，消息对话框的按钮和标签也被删掉了，这正是我们需要的，因为按钮和标签是消息对话框的子控件。

你也可以删除子对象，它们会被从父节点内移除。例如，当用户移除了一个工具栏，会导致删除它的一个`QToolBar`对象，在这种情况下，这个工具栏的父对象`QMainWindow`会检测到删除行为，并且会重新配置屏幕空间。

当应用程序看起来很奇怪，或者行为很奇怪的时候，调试函数`QObject::dumpObjectTree()`和`QObject::dumpObjectInfo()`是很有用的。

### `QObjects`的构造和析构顺序

当`QObjects`的对象在堆上创建是（例如使用`new`创建）,这棵树可以以任意的顺序被创建，而这棵树被销毁的时候，顺序也是任意的。当这棵树中的任意一个对象被删除时，如果它有父节点，析构函数会自动的将它从它的父节点中移除。如果这个这个对象有孩子节点，析构函数会将每一个孩子都析构掉。无论是什么顺序，没有`QObject`对象会被析构两次。

当`QObjects`对象被从栈上创建时，跟在堆上创建的行为相同。一般情况下，析构函数的顺序不会产生问题。考虑下面的代码片段：

```c++
int main()
{
    QWidget window;
    QPushButton quit("Quit", &window);
    ...
}
```

父控件`window`，和子控件 `quit`都是继承自`QObject`。这段代码是正确的。`quit`的析构函数不被被调用两次，因为C++标准规定，局部对象的析构函数的调用顺序是构造函数调用顺序的反顺序。因此，子控件`quit`的析构函数首先会被调用，并且它会将自己从父控件`window`中移除，该行为发生在`window`的析构函数被调用之前。

但是现在，考虑下面的代码片段会发生什么：

```c++
int main()
{
    QPushButton quit("Quit");
    QWidget window;

    quit.setParent(&window);
    ...
}
```

在这种情况下，析构函数的调用顺序会导致问题。父控件的析构函数先被调用，因为它的构造函数是后被调用的。然后它会调用子控件`quit`的析构函数，这样的顺序是不正确的，因为`quit`是一个局部变量。当`quit`离开这个作用域时，它的析构函数会被再次的调用，这回导致一些问题。