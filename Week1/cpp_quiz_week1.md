#! https://zhuanlan.zhihu.com/p/607978987
![image.png](https://s2.loli.net/2023/02/20/XHqTEx4LZzBSM7V.png)
# Cpp Quiz 练习记录 Week-1
图源P站`Mika Pikazo`.
## [Question #281](https://cppquiz.org/quiz/question/281) Difficulty: 2 

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

class C
{
public:
    C(){}
    C(const C&){} //User-declared, disables move constructor
};

int main()
{
    C c;
    C c2(std::move(c));
    std::cout << "ok";
}

```

hint:左右值。

### Answer

`ok`。可以编译成功。

### Explanation

> the xvalue `std::move(c)` is *both* an rvalue and a glvalue. So the clause "if [it] is an rvalue" above applies, as does "the resulting glvalue".
>
> So the `const C&` in the copy constructor is bound to the glvalue `std::move(c)`. The copy constructor is indeed a viable function, and is used to construct `c2`.

## [Question #125](https://cppquiz.org/quiz/question/125) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

using namespace std;

template <class T> void f(T) {
  static int i = 0;
  cout << ++i;
}

int main() {
  f(1);
  f(1.0);
  f(1);
}
```

### Answer：

`112`

### Explanation

因为不同类型的函数模板偏特化，会有自己独特的静态变量存储区。所以我们最终获得两个实例化的`f function` 。 

## [Question #188](https://cppquiz.org/quiz/question/188) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

int main() {
  char* a = const_cast<char*>("Hello");
  a[4] = '\0';
  std::cout << a;
}
```

### Answer

` The program is undefined.`

### Explanation

> Modifying string literals is undefined behavior. In practice, the implementation can for instance store the string literal in read-only memory, such as the code segment. Two string literals might even be stored in overlapping memory. So allowing them to be modified is clearly not a good idea.
>
> Also consider that the type of the string literal is array of const char, and thus not modifiable.

## [Question #251](https://cppquiz.org/quiz/question/251) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

template<class T>
void f(T) { std::cout << 1; }

template<>
void f<>(int*) { std::cout << 2; }

template<class T>
void f(T*) { std::cout << 3; }

int main() {
    int *p = nullptr; 
    f( p );
}
```

### Answer

`3`

解析：

谁更**专业**就匹配谁。

> The name `f` is overloaded by the two function templates `void f(T)` and `void f(T*)`.
>
> Note that overload resolution only considers the function templates, not the explicit specialisation `void f<>(int*)`! For overload resolution, first we deduce the template arguments for each function template, and get `T = int *` for the first one, and `T = int` for the second.
>
> Both function templates are viable, but which one is best? According to *[§[over.match.best\]¶1](https://timsong-cpp.github.io/cppwp/n4659/over.match.best#1)*, a function template is a better match than another function template if it's more specialised:



## [Question #276](https://cppquiz.org/quiz/question/276) Difficulty: 3

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

auto sum(int i)
{
  if (i == 1)
    return i;
  else
    return sum(i-1)+i;
}

int main()
{
    std::cout << sum(2);
}
```

### Answer

3.



## [Question #206](https://cppquiz.org/quiz/question/206) Difficulty: 3

According to the C++17 standard, what is the output of this program?

```
    #include <iostream>

int main() {
   int n = sizeof(0)["abcdefghij"]; 
   std::cout << n;   
}
```

### Answer

The program is guaranteed to output: `1`

### Explanation

We have several pieces of the puzzle, so let's peel away the layers.

unary-expression:
...

- sizeof unary-expression
- sizeof ( type-id )
- sizeof ... ( identifier )

... `sizeof(char)`, `sizeof(signed char)` and `sizeof(unsigned char)` are `1` ...



## [Question #132](https://cppquiz.org/quiz/question/132) Difficulty: 1

According to the C++17 standard, what is the output of this program?

```
    #include <iostream>
using namespace std;

int foo() {
  cout << 1;
  return 1;
}

void bar(int i = foo()) {}

int main() {
  bar();
  bar();
}
```

### Answer 

11.

## [Question #226](https://cppquiz.org/quiz/question/226) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>
#include <utility>

struct X {
    X() { std::cout << "1"; }
    X(X &) { std::cout << "2"; }
    X(const X &) { std::cout << "3"; }
    X(X &&) { std::cout << "4"; }
    ~X() { std::cout << "5"; }
};

struct Y {
    mutable X x;
    Y() = default;
    Y(const Y &) = default;
};

int main() {
    Y y1;
    Y y2 = std::move(y1);
}
```

### Answer

1255

### Explanation

嗯这题的话我也不太懂随便猜的，居然在第三次猜还猜对了。

首先的话，`multable`很重要，[详情见解析](https://liam.page/2017/05/25/the-mutable-keyword-in-Cxx/)，简单来说就是

> 显而易见，`mutable` 只能用来修饰类的数据成员；而被 `mutable` 修饰的数据成员，可以在 `const` 成员函数中修改。

注意，这是针对类中的`multable`的情况 。 Lambda 表达式中的 `mutable`同样也很有意思，这里不再赘述。

然后接着回到题目。

Y y1是默认构造，所以输出`1` 。 

接着y2使用了`move`的方法。这里本来是应该使用

```cpp
    X(const X &) { std::cout << "3"; }
```

然而`multable`让`X(X &) { std::cout << "2"; }`更加适合重载，因为在`multable`的修饰之下，`const`会变得无效。准确的说这里应该是编译器将把`x`理解成非常量`non-const`。

此外补充，`move`的时候运用到了这个[条款](https://timsong-cpp.github.io/cppwp/n4659/class.copy.ctor#14)

> The implicitly-defined copy/move constructor for a non-union class X performs a memberwise copy/move of its bases and members.

以上。

## [Question #198](https://cppquiz.org/quiz/question/198) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

namespace A {
  extern "C" { int x; }
};

namespace B {
  extern "C" { int x; }
};

int A::x = 0;

int main() {
  std::cout << B::x;
  A::x=1;
  std::cout << B::x;
}
```

### Answer

The program has a compilation error.

### Explanation

这题太妙了，很好的理解了一下`extern`这个关键字。

`extern`在多数情况下主要是在不同文件中指定同一个全局变量，[详见Stack overflow](https://stackoverflow.com/questions/10422034/when-to-use-extern-in-c)

不过在这里，它是指的`Language linkage`。比如说`extern "C"`就是说：

> "C", which makes it possible to link with functions written in the C programming language, and to define, in a C++ program, functions that can be called from the units written in C.
>
> 保证我们可以链接用C语言编写的函数，并且在C++程序定义可以从用C编写的单元调用的函数。

[关于语言链接的更多内容](https://en.cppreference.com/w/cpp/language/language_linkage)

我们可以推断出两个不同`namespace`里面的`x`其实是同一个。

紧接着，问题就是为什么会出现编译错误呢？

答案是，我们进行了重复的定义`definition`，这里需要重复一个很关键的概念，声明`declaration`和定义不同，且前者可以重复而后者不能。而在`namespace`里面，假设你把变量包含在大括号以内，那么很显然你是对这个变量进行了定义而不是声明。这是很关键的一点。*这里讲的不是很清楚，可以参考一下上文链接中的Special rules for "C" linkage*。

看个例子，比如说：

```cpp
int x;
 
namespace A
{
    extern "C" int x(); // error: same name as global-namespace variable x
    //错误，因为和全局变量有冲突，链接失效。
}
```

但是这样是可以的。

```cpp
namespace A
{
    extern "C" int f();
}
 
namespace B
{
    extern "C" int f();   // A::f and B::f refer to the same function f with C linkage
}
//注意这里声明的方式，没有花括号。
int A::f() { return 98; } // definition for that function
```

然而这里，就是题目里的错误了。

```cpp
namespace A
{
    extern "C" int g() { return 1; }
}
 
namespace B
{
    extern "C" int g() { return 1; } // error: redefinition of the same function
}
```

此外，还有一些内容存在`Notes`里面，我就不搬运上来了。比如说Language specifications can only appear in [namespace scope](https://en.cppreference.com/w/cpp/language/scope#Namespace_scope)，以及非常好的样例，可以自行阅读。

[IBM的文档](https://www.ibm.com/docs/zh/zos/2.2.0?topic=linkage-language-c-only)也解释了对这个设计的理解。可以参考阅读。

