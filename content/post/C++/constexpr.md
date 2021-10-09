# `constexpr` VS `const`

`constexpr`是C++11中新增的一个说明符，指明函数或者变量可以用在常量表达式中。有一点值得提一下，这个说明符的名字起得是不怎么样。其含义强调的是编译期求值，跟`const`的关系不大，但是很多人都会被迷惑。

举个例子说明一下，** `constexpr` 并没有常量特性**

```C++
#include <iostream>

const char* a = "abd";
int main()
{
    a[0] = 'c';
    return 0;
}
```

编译代码，会得到如下错误：

```bash
<source>: In function 'int main()':
<source>:6:10: error: assignment of read-only location '* a'
    6 |     a[0] = 'c';
      |     ~~~~~^~~~~
```



将上面的代码做一下修改

```c++
#include <iostream>

constexpr char* a = "abd";
int main()
{
    a[0] = 'c';
    return 0;
}
```

就可以顺利的编译运行。

在举例说明下 ** `constexpr`的编译期求值特性**

```C++
#include <iostream>

const int get(){return 4;}
int main()
{
    static_assert(4 == get());
    return 0;
}
```



编译代码，会得到如下错误

```bash
<source>: In function 'int main()':
<source>:6:21: error: non-constant condition for static assertion
    6 |     static_assert(4 == get());
      |                   ~~^~~~~~~~
<source>:6:27: error: call to non-'constexpr' function 'const int get()'
    6 |     static_assert(4 == get());
      |                        ~~~^~
<source>:3:11: note: 'const int get()' declared here
    3 | const int get(){return 4;}
      |           ^~~
```



再次修改代码

```c++
#include <iostream>

constexpr int get(){return 4;}
int main()
{
    static_assert(4 == get());
    return 0;
}
```

编译通过。