# [Question #283](https://cppquiz.org/quiz/question/283) Difficulty: ![2](https://cppquiz.org/static/level-2.png)

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

## Answer

210.

## Explanation

析构的顺序和构造的顺序相反。