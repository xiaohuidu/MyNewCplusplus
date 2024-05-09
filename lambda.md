lambda表达式的本质就是**重载了()运算符的类**，这种类通常被称为functor，即行为像函数的类。因此**lambda表达式对象其实就是一个匿名的functor**。

**C++中lambda表达式的构成**
一个标准的lambda表达式包括：**捕获列表、参数列表、mutable指示符、尾置返回类型（->返回类型）和函数体**：

**[capture list] (params list) mutable exception-> return type { function body }**

各项具体含义如下
 
1. capture list：捕获外部变量列表
2. params list：形参列表
3. mutable指示符：用来说用是否可以修改捕获的变量
4. exception：异常设定
5. return type：返回类型
function body：函数体
此外，我们还可以省略其中的某些成分来声明“不完整”的lambda表达式，常见的有以下几种：

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMzNTcwNTg1M119
-->