# Cpp Quiz 练习记录 Week2 Day1

嗯今天心情挺好的，学了一点点数学，但是数电没听懂来写点`Cpp_quiz`来解压。

## [Question #283](https://cppquiz.org/quiz/question/283) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>
class show_id
{
public:
    ~show_id() { std::cout << id; }
    int id;
};
int main()
{
    delete[] new show_id[3]{ {0}, {1}, {2} };
}
```

### Answer

210.

### Explanation

Easy.析构的顺序和构造的顺序相反。

## [Question #195](https://cppquiz.org/quiz/question/195) Difficulty:2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>
#include <cstddef>
#include <type_traits>

int main() {
  std::cout << std::is_pointer_v<decltype(nullptr)>;
}
```

### Answer

**0**

### Explanation

感觉应该不是`CE`，那么很显然不是`1`就是`0`，果断写了个`0`进去居然过了。

然后分析一下这是为什么。

首先`is_pointer_v`是`C++17`后启用的，对于它的解释是：

```cpp
template< class T >
inline constexpr bool is_pointer_v = is_pointer<T>::value;
```

那么这题就是判断`nullptr`是不是指针了。题后的解释告诉我们`nullptr`实际上不是指针，是`nullptr_t`类型的纯右值，它只是一个空指针常量，但它可以转化为指针。

附上英文解释更好理解一些！

> The pointer literal is the keyword `nullptr`. It is a prvalue of type `std::nullptr_­t`. [ Note: `std::nullptr_­t` is a distinct type that is neither a pointer type nor a pointer to member type; rather, a prvalue of this type is a null pointer constant and can be converted to a null pointer value or null member pointer value. See [conv.ptr] and [conv.mem].  — end note ]

## [Question #120](https://cppquiz.org/quiz/question/120) Difficulty:1

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

int main() {
  int a = 10;
  int b = 20;
  int x;
  x = a, b;
  std::cout << x;
}
```

### Answer

20

### Explanation

逗号的优先级很低，只会执行`x=a`。

## [Question #339](https://cppquiz.org/quiz/question/339) Difficulty:2

According to the C++17 standard, what is the output of this program?

```cpp
#include <future>
#include <iostream>

int main()
{
    std::promise<int> p;
    std::future<int> f = p.get_future();
    p.set_value(1);
    std::cout << f.get();
    std::cout << f.get();
}
```

### Answer

The program is undefined.

### Explanation

`future`只能让你`get value`一次，如果多次调用是个`UB`（但是通常是会抛出异常的，`GCC`,`Clang`和`MSVC`的大多数版本都是如此。）

深入一点，当我们第一次调用`get`的时候，会一直`wait()`直到`shared state is ready`.最终我们获取到了值。然后我们将共享状态释放掉(`release`)。

那么当我们最终得到返回对象的时候，意味着放弃了对`共享状态的引用`。

我们再一次地调用`get`的时候，*[§[futures.unique_future\]¶3](https://timsong-cpp.github.io/cppwp/n4659/futures.unique_future#3)* says:

> The effect of calling any member function other than the destructor, the move-assignment operator, `share`, or `valid` on a `future` object for which `valid() == false` is undefined. 

所以是`UB`。

## [Question #307](https://cppquiz.org/quiz/question/307) Difficulty:2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

struct S
{
  S() = delete;
  int x;
};

int main()
{
  auto s = S{};
  std::cout << s.x;
}
```

### Answer

0

### Explanation

一遍过！(我承认这题有赌的成分。

不过看解析的时候我有点懵。因为我不太了解`aggregate `这个东西。所以我找了点资料去。

[看点赞最高的老哥的回答就行了](https://stackoverflow.com/questions/4178175/what-are-aggregates-and-pods-and-how-why-are-they-special)

我摘取一些重点。

让我们从定义开始！

**Formal definition from the C++ standard (\*C++03 8.5.1 §1\*)**:

> An aggregate is an array or a class (clause 9) with no user-declared constructors (12.1), no private or protected non-static data members (clause 11), no base classes (clause 10), and no virtual functions (10.3).

老哥补充了一些重点。

> What do these criteria imply?
>
> - This does not mean an aggregate class cannot have constructors, in fact it can have a default constructor and/or a copy constructor as long as they are implicitly declared by the compiler, and not explicitly by the user.**非显式声明的构造即可**
> - No private or protected ***non-static data members***. You can have as many private and protected member functions (but not constructors) as well as as many private or protected ***static*** data members and member functions as you like and not violate the rules for aggregate classes。**不能拥有私有或者受保护的非静态数据成员**
> - An aggregate class can have a user-declared/user-defined copy-assignment operator and/or destructor.**聚合类可以具有用户声明/用户定义的复制赋值运算符和/或析构函数**
> - An array is an aggregate even if it is an array of non-aggregate class type. **数组是聚合**

其次，我们只用花括号来初始化聚合类，初始化是一一对应的，即，我们将按照非静态数据成员在类定义中出现的顺序来初始化非数组元素。考虑两种情况：

- 假设我们不能一一对应初始化（初始化列表`initializers`中的数量少于本应该有的成员数量），那么我们考虑进行将其余的值，进行默认初始化。若对象不支持默认初始化，那么这个时候编译错误。
- 初始化列表`initializers`中的数量多于本应该有的成员数量也会编译错误。

现在我们知道了聚合的特别之处，更多的，我们在这里了解聚合和类的区别。很显然，用大括号进行成员初始化意味着该类只不过是其成员的总和。如果存在用户定义的构造函数，则意味着用户需要做一些额外的工作来初始化成员，而聚合不想这么繁琐，因此大括号初始化是不正确的。

不过，标准中提到的`no virtual functions (10.3) 不支持虚函数`是怎么回事？我们思考类中假如有虚函数的话，那么它的实现，往往需要一个指向类的所谓 `vtable`虚函数表的指针，很显然这不符合我们对于聚合的要求，也意味着我们很难去对聚合初始化。

至于为什么不允许基类，也是同理，可以自行思考。

如果只是做题的话，到这里应该已经足够了，我们知道`Struct S`中的构造函数被禁用了，那么在用花括号初始化`x`为`0`很自然。

在这里我的理解是，聚合可以让那些简单类的初始化更加简单和直观，毕竟有时候我们不希望类太复杂，这个时候初始化会很容易！

不过在`C++20`中，这种行为会出现编译错误，理由是这个时候`S`不是聚合，它被认为在`S() = delete`已经是被用户提供了一个构造函数。

如果有更多的关于这方面的内容也欢迎补充，我只是才学这个的小白qwq.

上面的链接里讲的更加详细，也可以作为参考，这里没有介绍更多细节。







