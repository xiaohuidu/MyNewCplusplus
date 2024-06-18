### discarded-value 表达式
一个discarded-value 表达式是一个只利用其副作用(side-effect)的表达式， 表达式计算出来的值被丢弃。
discarded-value 表达式包括:

 - 全的表达式语句。
 - 内置的逗号(,) 操作符的做边操作数 a, b 
 - 被转化成 void 类型的 cast 表达式的操作数

数组到指针的转换， 函数到指针的转换 不能引用在 discarded-value 表达式计算出来的值。

###  [[nodiscard]]
在 C++ 中，`[[nodiscard]]` 属性用于指示函数的返回值不应被忽略。
当调用带有 `[[nodiscard]]` 标记的函数并且其返回值被丢弃而没有使用时，编译器会发出警告。这有助于防止潜在的错误，例如返回值被意外忽略。

```cpp
struct [[nodiscard]] error_info { /*...*/ };
 
error_info enable_missile_safety_mode() { /*...*/ return {}; }
 
void launch_missiles() { /*...*/ }
 
void test_missiles()
{
    enable_missile_safety_mode(); // compiler may warn on discarding a nodiscard value
    launch_missiles();
}
 
error_info& foo() { static error_info e; /*...*/ return e; }
 
void f1() { foo(); } // nodiscard type is not returned by value, no warning
 
// nodiscard( string-literal ) (since C++20):
[[nodiscard("PURE FUN")]] int strategic_value(int x, int y) { return x ^ y; }
 
int main()
{
    strategic_value(4, 2); // compiler may warn on discarding a nodiscard value
    auto z = strategic_value(0, 0); // OK: return value is not discarded
    return z;
}
```
输出结果:
```
game.cpp:5:4: warning: ignoring return value of function declared with ⮠
 'nodiscard' attribute
game.cpp:17:5: warning: ignoring return value of function declared with ⮠
 'nodiscard' attribute: PURE FUN
 ```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUyNjg2NDQ2NCwxODM3NDg0NzYwLDI5Nj
AyMTYzMF19
-->