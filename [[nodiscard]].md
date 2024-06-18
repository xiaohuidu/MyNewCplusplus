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

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjkyMjQ4MjgyLDE4Mzc0ODQ3NjAsMjk2MD
IxNjMwXX0=
-->