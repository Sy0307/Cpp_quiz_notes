#! https://zhuanlan.zhihu.com/p/608339877
![image.png](https://s2.loli.net/2023/02/21/F2mfgqVvxEshbBz.png)
# Cpp Quiz 练习记录 Week2 Day2

图源P站`碧風羽`

今天考研出分了的说，希望大家都能获得理想的成绩呀！

感觉自己的心理问题太严重了，一直在焦虑，希望自己能沉下心来做事吧！

## [Question #248](https://cppquiz.org/quiz/question/248) Difficulty:

According to the C++17 standard, what is the output of this program?

```cpp
#include <algorithm>
#include <iostream>

int main() {
    int x = 10;
    int y = 10;

    const int &max = std::max(x, y);
    const int &min = std::min(x, y);

    x = 11;
    y = 9;

    std::cout << max << min;
}
```

### Answer

**1111**

### Explanation

猜测了一下`std::max`和`std::min`比较函数应该是`大于等于`和`小于等于`.

果然如此，那么返回第一个参数即可。最终我们两个引用都指向了变量`x`.

>The C++17 standard says both for `std::min()` in *[§[alg.min.max\]¶3](https://timsong-cpp.github.io/cppwp/n4659/alg.min.max#3)* and for `std::max()` in *[§[alg.min.max\]¶11](https://timsong-cpp.github.io/cppwp/n4659/alg.min.max#11)*:
>
>​	Returns the first argument when the arguments are equivalent.

虽然这题很简单，但是可以讲下去的东西很多，解析里面额外提到了

> Some say it would be better if `min()` returned its first element, but `max()` returned its *last* element. Sean Parent explains his rationale for this in his BoostCon 2016 Keynote [Better Code](https://www.youtube.com/watch?v=giNtMitSdfQ&t=1448s).

这个里面的链接我看了一小段，关于这个的解释，讲的很好。大佬说这样设计是为了更加直观，假设相等是返回第二个元素，那么当一堆数比较时你很难去定义你找到的元素的大小关系。但是如果倾向于第一个元素，那么可以保证你找的是最前面的，更简单直观一点。我觉得这样的设计还不错，感兴趣的也可以看看[演讲材料](https://github.com/boostcon/cppnow_presentations_2016).

## [Question #152](https://cppquiz.org/quiz/question/152) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>
#include <type_traits>
 
int main() {
    if(std::is_signed<char>::value){
        std::cout << std::is_same<char, signed char>::value;
    }else{
        std::cout << std::is_same<char, unsigned char>::value;
    }
}
```

### Answer

0

## [Question #161](https://cppquiz.org/quiz/question/161) Difficulty:3

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

int main() {
    int n = 3;
    int i = 0;

    switch (n % 2) {
    case 0: 
    do {
    ++i;
    case 1: ++i;
    } while (--n > 0);
    }

    std::cout << i;
}
```

### Answer

5

### Explanation

虽然答对了但是是猜出来的，感觉这段代码很诡异。

然后解答里提到了[Duff's device](https://en.wikipedia.org/wiki/Duff%27s_device).

不过[这篇回答](https://stackoverflow.com/questions/514118/how-does-duffs-device-work)说的还可以，此外你可以把这些复杂的循环当成简单的`goto`即可。

## [Question #12](https://cppquiz.org/quiz/question/12) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

int main() {
  static int a;
  std::cout << a;
}
```

### Answer

0

## [Question #244](https://cppquiz.org/quiz/question/244) Difficulty:3

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

using namespace std;

template <typename T>
struct A {
    static_assert(false);
};

int main() {
    cout << 1;
}
```

### Answer 

UB.

### Explanation

通常来说，`static_assert(false)`会导致编译失败，但是我们并没有实例化这个模板，编译器无法确定是忽略还是报错，所以UB。

不过，需要注意的是，经过测试，最新的 gcc、clang 和 msvc 都给出了此示例的编译错误。所以正确答案应该是CE.

## [Question #187](https://cppquiz.org/quiz/question/187) Difficulty: 2

According to the C++17 standard, what is the output of this program?

```cpp
#include <iostream>

struct C {
  C() { std::cout << "1"; }
  C(const C& other) { std::cout << "2"; }
  C& operator=(const C& other) { std::cout << "3"; return *this;}
};

int main() {
  C c1;
  C c2 = c1;
}
```

### Answer

12

### Explanation

首先`c1`的初始化没有什么好说的。我们关注`c2`这个时候只是进行了一次所谓的`复制初始化`，虽然出现了`=`，但是实际上并没有运用这个重载，只是单纯通过第二种方式初始化了一下。