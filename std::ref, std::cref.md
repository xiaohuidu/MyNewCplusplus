解释
std::ref 用于包装按引用传递的值。
std::cref 用于包装按const引用传递的值。

为什么需要std::ref和std::cref
bind()是一个函数模板，它的原理是根据已有的模板，生成一个函数，但是由于bind()不知道生成的函数执行的时候，传递进来的参数是否还有效。所以它选择参数值传递而不是引用传递。如果想引用传递，std::ref和std::cref就派上用场了。
```
#include <functional>
#include <iostream>

void f(int& n1, int& n2, const int& n3)
{
    std::cout << "In function: n1[" << n1 << "]    n2[" << n2 << "]    n3[" << n3 << "]" << std::endl;
    ++n1; // 增加存储于函数对象的 n1 副本
    ++n2; // 增加 main() 的 n2
    //++n3; // 编译错误
    std::cout << "In function end: n1[" << n1 << "]     n2[" << n2 << "]     n3[" << n3 << "]" << std::endl;
}

int main()
{
    int n1 = 1, n2 = 1, n3 = 1;
    std::cout << "Before function: n1[" << n1 << "]     n2[" << n2 << "]     n3[" << n3 << "]" << std::endl;
    std::function<void()> bound_f = std::bind(f, n1, std::ref(n2), std::cref(n3));
    bound_f();
    std::cout << "After function: n1[" << n1 << "]     n2[" << n2 << "]     n3[" << n3 << "]" << std::endl;
}
```

运行结果：
```
Before function: n1[1]     n2[1]     n3[1]
In function: n1[1]    n2[1]    n3[1]
In function end: n1[2]     n2[2]     n3[1]
After function: n1[1]     n2[2]     n3[1]
```

分析
n1是值传递，函数内部的修改对外面没有影响。
n2是引用传递，函数内部的修改影响外面。
n3是const引用传递，函数内部不能修改。

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIyMzg2ODIxNiw3MzA5OTgxMTZdfQ==
-->