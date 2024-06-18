### discarded-value 表达式
一个discarded-value 表达式是一个只利用其副作用(side-effect)的表达式， 表达式计算出来的值被丢弃。
discarded-value 表达式包括:

 - 全的表达式语句。
 - 内置的逗号(,) 操作符的做边操作数 a, b 
 - 被转化成 void 类型的 cast 表达式的操作数

数组到指针的转换， 函数到指针的转换 不能引用在 discarded-value 表达式计算出来的值。

###  [[nodiscard]]
如果一个声明了 nodiscard的函数， 

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE2NDgzMTUyMywxODM3NDg0NzYwLDI5Nj
AyMTYzMF19
-->