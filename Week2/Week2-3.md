# Cpp Quiz Record Week2 Day3

最近感觉有点怠惰，刚好发现了`Rust`的一些好玩的项目，感觉可以试试看。

## [Question #160](https://cppquiz.org/quiz/question/160) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

struct A {
    virtual void foo (int a = 1) {
        std::cout << "A" << a;
    }
};

struct B : A {
    virtual void foo (int a = 2) {
        std::cout << "B" << a;
    }
};

int main () {
    A *b = new B;
    b->foo();
}
```

### Answer

B1

### Explanation

考察了虚函数一个很有意思的点，虽然在有虚函数的情况下找到的自身的同名函数，也就是说这里调用的是`B::foo()`，但是默认参数还是使用虚函数声明中的默认参数。对于派生类的覆盖函数中，我们不会从它覆盖的函数中获取默认参数。

原文如下：

>However, which default argument is used for the `int a` parameter to `foo()`? *[§[dcl.fct.default\]¶10](https://timsong-cpp.github.io/cppwp/n4659/dcl.fct.default#10)* in the standard:
>
>​	A virtual function call (§ 13.3) uses the default arguments in the declaration of the virtual function determined by the static type of the pointer or reference denoting the object. An overriding function in a derived class does not acquire default arguments from the function it overrides.

## [Question #233](https://cppquiz.org/quiz/question/233) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <type_traits>
#include <iostream>

using namespace std;

struct X {
    int f() const&&{
        return 0;
    }
};

int main() {
    auto ptr = &X::f;
    cout << is_same_v<decltype(ptr), int()>
         << is_same_v<decltype(ptr), int(X::*)()>;
}
```

### Answer

00

### Explanation

`ptr`的类型是``int(X::*)() const&&``,我们需要记住的是，以下的是函数类型的一部分：

- 返回类型
- 参数类型列表
- 参数限定符
- cv限定符（`const` 和 `volatile`）
- 异常规定

所以在判断的时候需要仔细对照一下就行。

## [Question #281](https://cppquiz.org/quiz/question/281) Difficulty: 1

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

### Answer

ok

### Explanation

虽然难度是1，但是我感觉有点难的说。

首先因为我们声明了一个复制构造函数，所以我们的隐式移动构造函数就不存在了。

那么问题来了，我们没有移动构造函数，但是这个时候我们还能初始化`c2`嘛？

答案是可以的，根据[这篇博客](https://blog.knatten.org/2018/03/09/lvalues-rvalues-glvalues-prvalues-xvalues-help/)里面的说法，`std::move(c)`最终又是左值又是右值，最终复制函数可以绑定到`glvalue std::move(c)`，我们成功构造出了`c2`！

## [Question #273](https://cppquiz.org/quiz/question/273) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

struct A {
    A() { std::cout << "A"; }
    ~A() { std::cout << "a"; }
};

int main() {
    std::cout << "main";
    return sizeof new A;
}
```

### Answer

main

### Explanation

首先没有编译错误，输出`main`.

那么我们关注`sizeof`,对于`sizeof`我们期望它的操作数是以下三种之一：

- 一个表达式（一个未评估的操作数）
- 一个带括号的类型标识

通常来说我们使用的是第二种，比如说`sizeof(int)`，但是在题目中我们使用的是`sizeof expression`的形式，也就是第一种。

未评估在这里就是未执行的意思，很显然我们不会构造这个`A`出来，顺理成章我们也不会对它进行析构。

## [Question #333](https://cppquiz.org/quiz/question/333) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

int main()
{
    int x = 3;
    while (x --> 0) // x goes to 0
    {
        std::cout << x;
    }
}
```

### Answer

210

### Explanation

以一个简单的题来结束辛苦的一天！