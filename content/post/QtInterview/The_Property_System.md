---
title: "Qt属性系统"
date: 2020-08-29T17:43:10+08:00
draft: false
---

# Qt 属性系统

[The Property System | Qt Core 6.2.0](https://doc.qt.io/qt-6/properties.html)

Qt 提供了严谨的属性系统，这与一些编译器所提供的属性系统类似。但是，作为一个编译器无关、平台无关的第三方库，Qt不会依赖非标准的编译器特性，如`__property` 或者` [property]`。Qt所提供的解决方案能够工作在任何Qt所支持的平台编译器。其基于Qt的元对象系统，该系统也提供了信号和槽用于对象间的通讯。

### 属性声明的要求

为了声明一个属性，需要在继承`QObject`的类中使用宏`Q_PROPERTY()`。

```c++
Q_PROPERTY(type name
           (READ getFunction [WRITE setFunction] |
            MEMBER memberName [(READ getFunction | WRITE setFunction)])
           [RESET resetFunction]
           [NOTIFY notifySignal]
           [REVISION int | REVISION(int[, int])]
           [DESIGNABLE bool]
           [SCRIPTABLE bool]
           [STORED bool]
           [USER bool]
           [BINDABLE bindableProperty]
           [CONSTANT]
           [FINAL]
           [REQUIRED])
```

下面是一些属性声明的典型例子，这些属性来自`QWidget`。

```c++
Q_PROPERTY(bool focus READ hasFocus)
Q_PROPERTY(bool enabled READ isEnabled WRITE setEnabled)
Q_PROPERTY(QCursor cursor READ cursor WRITE setCursor RESET unsetCursor)
```

下面的例子用来演示如何使用`MEMBER`关键字将变量导出成Qt 属性。注意，`NOTIFY`信号必须被指定允许QML 属性绑定。

```c++
Q_PROPERTY(QColor color MEMBER m_color NOTIFY colorChanged)
Q_PROPERTY(qreal spacing MEMBER m_spacing NOTIFY spacingChanged)
Q_PROPERTY(QString text MEMBER m_text NOTIFY textChanged)
signals:
    void colorChanged();
    void spacingChanged();
    void textChanged(const QString &newText);

private:
    QColor  m_color;
    qreal   m_spacing;
    QString m_text;
```
属性的行为跟类的数据成员很像，但是其有一些额外的属性可以通过元对象系统访问。

* 在没有`MEMBER`来指定变量时，`READ`函数不能被省略。只有这样才能读取属性值。理想情况下，一个常量函数被用于这个目的，并且这个函数的返回类型为属性的类型或者为这个类型的常量引用。例如，`QWidget::focus`是一个只读属性，其`READ` 函数为 `QWidget::hasFocus()`。
* `WRITE`函数是可选的。其作用是设置属性的值。它的返回类型必须为`void`，并且只能有一个参数，参数类型或者是属性的类型，或者是对该类型的指针或引用。例如，`QWidget::enabled`有`WRITE` 函数`QWidget::setEnabled()`. 只读属性不需要`WRITE`函数，例如`QWidget::focus` 没有 `WRITE` 函数。
* 如果没有`READ`函数，那么`MEMBER`变量是不能省略的。这一点可以使得，在没有`READ`和`WRITE`函数的情况下，给定的成员变量可读和可写。但是在这种情况下仍然可以有`READ`和`WRITE`函数，这样可以弥补成员变量没有设置或者获取变量值的问题。
* `RESET`函数是可选的。该函数是为了将属性的值设置为上下文指定的默认值。例如，`QWidget::cursor`有`READ` 和`WRITE` 函数， `QWidget::cursor()` 和 `QWidget::setCursor()`, 并且，它也有`RESET` 函数`QWidget::unsetCursor()`，如果不调用 `QWidget::setCursor()`函数，这意味着光标能够被重置为上下文指定的光标。`RESET`函数必须返回`void`并且没有参数。
* `NOTIFY`信号是可选的。如果该信号被定义了，它应该指定一个类中已经存在的信号，该信号会在属性值被改变的时候发出。对于`MEMBER` 变量的`NOTIFY`信号有至多一个参数，其类型跟属性的类型相同。这个参数的值为属性被更改后的新值。`NOTIFY`信号应当只有在值**真正**被改变的时候才被发出，从而避免被无意义的重复求值，例如在QML中。当被没有显示赋值函数的`MEMBER`属性需要的时候Qt会自动的发出这个信号 。
* `REVISION `数字或者宏`REVISION() `是可选的。默认值为0。其作用是为了限制其通知信号只在特定的版本可以被使用。
* `DESIGNABLE`是为了表明该属性是否应该在GUI的设计工具的属性编辑器中显示出来（例如，Qt Designer）。大多数的属性是`DESIGNABLE `的（其默认值为`true`）。有效的值是`true`或者`false`。
* `SCRIPTABLE`表明这个属性是否可以被脚本引擎获取（默认值为`true`）。有效值是`true`或者`false`。
* `STORED`表明这个属性是否应该被认为是独立的，或者是依赖其他的值。它也表明当存储对象的状态时，这个属性值是否会被保存。大多数属性是`STORED`（默认值为`true`）。但是`QWidget::minimumWidth()`的`STORED`为`false`，因为它的值仅仅是`QWidget::minimumSize()`属性（其类型为`QSize`）的宽度分量。
* `USER`表明这个属性是否是用户可编辑的或者用户看到的。一般情况下，每个类只有一个`USER`属性（默认值为`false`）。例如，对于（可以选中的）按钮来说，`QAbstractButton::checked`是一个用户可编辑的属性， is the user editable property for (checkable) buttons. Note that `QItemDelegate` 用来设置或者读取widget的`USER`属性。
* `BINDABLE`表明该属性支持绑定，其可能通过元对象系统（`QMetaProperty`）设置或者检查绑定。可绑定属性在类中的类型为`QBindable<T>`，其中，`T`是属性的类型。该属性由Qt 6.0 引入。
* `CONSTANT `属性表明属性值是一个常量。对于给定对象实例，每一个调用常量属性的`READ`方法，都应该返回一个相同的值。这个常量值可能因为对象的不同而不同。`CONSTANT`属性不能有`WRITE`方法和`NOTIFY`信号。
* `FINAL`表明该属性不能被子列覆盖。其用处是在某些情况下进行性能优化，但是不被moc强制。必须注意的是，永远不用覆盖`FINAL`属性。
* `REQUIRED`表明该属性必须被类的用户所赋值。moc对此是强制的，并且大多数情况下用于类暴露给QML。在QML中，有`REQUIRED`属性的类不能被实例化，除非所有的`REQUIRED`属性都被设置。

`READ`，` WRITE` 和 `RESET`函数都是可以被继承的。他们可以是虚函数。当用于多继承的情况时，该属性来自于第一个基类中继承的类。

属性的类型可以是任何被`QVariant`支持的类型。或者是用户定义的类型。在下面的例子中，类`QDate`被认为是一个用户自定义的类型。

```c++
Q_PROPERTY(QDate date READ getDate WRITE setDate)
```

由于`QDate`是用户自定义的，你必须包含头文件`<QDate>`用于属性的声明。



For historical reasons, QMap and QList as property types are synonym of QVariantMap and QVariantList.

### 使用元对象系统来写和读属性

在只知道属性名字的情况下，可以通过通用的函数` QObject::property()`和` QObject::setProperty()`来读取和设置属性的值。在下面的代码中，`QAbstractButton::setDown()` 和`QObject::setProperty()`的作用是一样的，都是设置属性`down`的值。 

```c++
QPushButton *button = new QPushButton;
QObject *object = button;

button->setDown(true);
object->setProperty("down", true);
```

使用`WRITE`访问属性要比上面的两种方法要好，因为其速度快，且在编译器能够有更好的诊断信息，但是，使用这种方法需要确切的在编译期知道属性所在的类的类型。通过名字访问属性，不必在编译期知道类的类型。你可以通过询问属性的`QObject`，`QMetaObject`和`QMetaProperties`来找到属性。

```c++
QObject *object = ...
const QMetaObject *metaobject = object->metaObject();
int count = metaobject->propertyCount();
for (int i=0; i<count; ++i) {
    QMetaProperty metaproperty = metaobject->property(i);
    const char *name = metaproperty.name();
    QVariant value = object->property(name);
    ...
}
```

在上面的代码片段中，`QMetaObject::property()`被用来获取定义在未知类型的类中的属性。属性名称被从元数据中取出来，并且传给`QObject::property()`来获取当前对象该属性的值。

### 一个简单的例子

假设我们有一个类`MyClass`，这个类继承自`QObject`，并且在类的私有区域使用了宏`Q_OBJECT`。我们想要在`MyClass`中声明一个属性来跟踪优先级。这个属性的名字就叫做`priority`，并且它的类型是一个定义在`MyClass`中的枚举类型`Priority`。

我们在私有区域使用宏`Q_PROPERTY()`来声明一个属性。`READ`函数被命名为`priority`，`WRITE`函数被命名为`setPriority`。枚举类型必须使用宏`Q_ENUM()`注册到元对象系统中。注册这个枚举类型可以使得该枚举类型在`QObject::setProperty()`中被调用。我们也必须提供`READ`和`WRITE`函数的声明。类`MyClass`看起来如下：

```c++
class MyClass : public QObject
{
    Q_OBJECT
    Q_PROPERTY(Priority priority READ priority WRITE setPriority NOTIFY priorityChanged)

public:
    MyClass(QObject *parent = nullptr);
    ~MyClass();
    enum Priority { High, Low, VeryHigh, VeryLow };
	Q_ENUM(Priority)

	void setPriority(Priority priority)
	{
    	m_priority = priority;
    	emit priorityChanged(priority);
	}
	Priority priority() const
	{ return m_priority; }
signals:
    void priorityChanged(Priority);

private:
    Priority m_priority;
};
```

`READ`函数是一个常量函数，并且返回类型为属性类型。`WRITE`函数返回空，且有一个类型为属性类型的参数。`moc`会强制检查这些要求。

考虑一个指向`MyClass`实例的指针，或者一个指向`QObject`的指针（是一个`MyClass`实例），我们有两种方式来设置它的`Priority`属性值：

```c++
MyClass *myinstance = new MyClass;
QObject *object = myinstance;

myinstance->setPriority(MyClass::VeryHigh);
object->setProperty("priority", "VeryHigh");
```

在这个例子中，作为属性类型的枚举类型在`MyClass`中被声明，且使用宏`Q_ENUM()`被注册到元对象系统中。这使得枚举值可以作为一个字符串在调用`setProperty()`的时候被使用。如果这个枚举类型被声明在其他类中，它要求使用完整的名称（例如：`OtherClass::Priority`），并且该其他类必须继承自`QObject`，并且使用宏`Q_ENUM()`进行了注册。

一个类似的宏，`Q_FLAG()`也是可用的。它注册一个枚举类型，该枚举类型的值时一系列的标志，这些标志可以使用`OR`（或）来进行运算。一个I/O类可能会有`Read`和`Write`值，那么`Read|Write`可以作为`QObject::setProperty()`的参数。`Q_FLAG()`应该被用来注册这个枚举类型。

### 动态属性

`QObject::setProperty() `也可以在运行时给一个类的实例添加新的属性。当该函数以某个名字和某个值为参数被调用时，如果存在一个与该名字一样的属性，并且传入的值的类型与该属性的值的类型一致，该值就会被存储在该属性中，且函数返回值为`true`。如果值与属性的类型不匹配，属性不会被更改，且返回值为`false`。如果该名字在`QObject`不存在的话（例如，并没有使用`Q_PROPERTY()`被声明），一个以函数参数中的名字作为名字，以参数中的值作为值的新属性被添加到`QObject`，但此时仍然返回为`false`。这意味着返回`false`并不能用来判断一个特定的属性是否真的被设置，除非你提前知道这个属性已经存在与`QObject`。

需要注意的是，动态属性会被加入到每一个实例中，例如它们被加入到`QObject`中，而不是`QMetaObject`。一个属性可以通过使用无效的`QVariant` 作为参数调用`QObject::setProperty()`被从一个实例中移除。`QVariant `的默认构造函数会生成一个无效的`QVariant`。

动态属性可以通过`QObject::property()`查询，跟在编译期使用`Q_PROPERTY()`声明的属性一致。

### 属性和客制化类型

如果客制化类型想要在属性中被使用，该类型需要使用宏`Q_DECLARE_METATYPE()`来注册，以便该值可以被存储在`QVariant `中。这样，客制化类型既可以在编译期使用宏`Q_PROPERTY()`来声明，也可以运行时作为动态属性的值的类型。

### 给类添加额外的信息

可以通过宏`Q_CLASSINFO()`，来给类的元对象附加额外的名字-值对。

```c++
Q_CLASSINFO("Version", "3.0.0")
```

像其他的元数据，类的信息都可以在运行期通过元对象获得。

