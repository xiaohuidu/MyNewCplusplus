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
6. function body：函数体
此外，我们还可以省略其中的某些成分来声明“不完整”的lambda表达式，常见的有以下几种：

|序号| 格式 |
|--|--|
| 1 |[capture list] (params list) -> return type {function body}  |
| 2 |[capture list] (params list) {function body}  |
| 3 |[capture list] {function body}  |

其中：

格式1声明了const类型的表达式，这种类型的表达式不能修改捕获列表中的值。
格式2省略了返回值类型，但编译器可以根据以下规则推断出Lambda表达式的返回类型：
如果function body中存在return语句，则该Lambda表达式的返回类型由return语句的返回类型确定
如果function body中没有return语句，则返回值为void类型。
格式3中省略了参数列表，类似普通函数中的无参函数。
## 简单lambda表达式及其原理分析

在VS2017中构造一个简单的lambda表达式如下：
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU5ODE0MDU3OSwzODU4MDk2MjIsOTQ2NT
M0NzUzLDE1OTE0MzQwMzddfQ==
-->