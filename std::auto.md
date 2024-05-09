## 一、std::auto发展历程

在早期C/C++中auto的含义是：使用auto修饰的变量，是具有自动存储器的局部变量，但遗憾的是一直没有人去使用它，大家可思考下为什么？

```cpp
int a = 10 ;  //拥有自动生命期
auto int b = 20 ;//拥有自动生命期
static int c = 30 ;//延长了生命期

```

C++11中，标准委员会赋予了auto全新的含义即：auto不再是一个存储类型指示符，而是作为一个新的类型指示符来指示编译器，**auto声明的变量必须由编译器在编译时期推导而得**。

## 二、auto 原理

auto的自动类型推断发生在编译期，所以使用auto并不会造成程序运行时效率的降低。而是否会造成编译期的时间消耗，我认为是不会的，在未使用auto时，编译器也需要得知右操作数的类型，再与左操作数的类型进行比较，检查是否可以发生相应的转化，是否需要进行隐式类型转换。  

需要注意的是，auto不是一个类型的“声明”，而是一个“占位符”，编译器在编译期会将auto替换为变量实际的类型。**使用auto定义变量时必须对其进行初始化**，在编译阶段编译器需要根据初始化表达式来推导auto的实际类型。它自动推导变量类型是根据“=”右侧的变量类型决定的。

auto使用的是**模板实参推断**（Template Argument Deduction）的机制。auto被一个虚构的模板类型参数T替代，然后进行推断，即相当于把变量设为一个函数参数，将其传递给模板并推断为实参，auto相当于利用了其中进行的实参推断，承担了模板参数T的作用。  

```cpp
std::vector<int> v{1,2,3,4,5,6};
auto pos = v.begin();

//auto的初始化相当于下面这个模板传参时的情形，T就是为auto推断的类型

template<typename T>
void deducePos(T pos);
deducePos(v.begin());
```

而**auto类型变量不会是引用类型**（模板实参推断的规则），所以要用auto&_（**C++14支持直接用decltype(auto)推断原始类型**）_，第二个auto推断对应于下面这个模板传参时的情形，同样T就是为auto推断的类型,例如：

```cpp
std::vector<int> v{1,2,3,4,5,6};
auto pos = v.begin();
auto& element = *pos++;

// auto& element = *pos++的推断等价于如下调用模板的推断
template<typename T>
void deduceElement(T& element);
deduceElement(*pos++);
```

  

## 三、使用方法和规则

**为什么使用auto？**

对于某些较长或较奇怪的数据类型，可交给编译器自行推导，这样使代码更简洁。

### 1.auto 使用方法简单示例

```cpp
#include <iostream>

int main()
{
	int a = 8;
	char b = 'x';
	auto* c = &a;
	auto& d = b;

	std::cout << typeid(a).name() << std::endl;
	std::cout << typeid(b).name() << std::endl;
	std::cout << typeid(c).name() << std::endl;
	std::cout << typeid(d).name() << std::endl;
}

// 输出
int
char
int *
char
```

### 2.auto  和const

首先顶层const和底层const的说法和指针有很大的关系。指针本身是一个对象，但它又指向另一个对象，这就是指针的两种属性。

**顶层const(high-level const) 和底层const（low-level const）**

对于指针本身是一个常量，即指针的指向是一个常量，也就说不能改变指针的指向，称其为顶层const属性；

对于指针指向的对象是一个常量，即指针指向的地址的值是一个常量，也就是说不能改变指针指向内存的值，称其为底层 const属性。

在代码表现上是：

```cpp
int* const p1 = &a;//p1是顶层const
const int* p2 = &a;//p2是底层const
```

**复合类型compound type、const和auto的关系是什么？**

编译器为auto推断的类型并不总是与初始化器的类型完全相同。相反，编译器调整类型以符合正常的初始化规则。

首先，**auto通常忽略引用**。正如我们所看到的，当我们使用引用时，我们实际上是在使用引用所指向的对象。

特别是，当使用引用作为初始化器时，初始化器是相应的对象。编译器使用该对象的类型进行auto的类型推导:

其次，**auto通常忽略顶层const**。在**初始化中，通常会保留低级const**，例如当初始化器是指向const对象的指针时，将被保留：

```cpp
int i = 0, &r = i;
const int ci = i, &cr = ci;
auto b = ci; // b is an int (top-level const in ci is dropped)
auto c = cr; // c is an int (cr is an alias for ci whose const is top-level)
auto d = &i; // d is an int*(& of an int object is int*)
auto e = &ci; // e is const int*(& of a const object is low-level const)
auto f = &cr; // e is const int*(& of a const object is low-level const)
```

下面看忽视顶层const指针的例子：

```cpp
int z = 9,z1 = 10;

int* const pz1 = &z;//pz1为int* const(顶层const)
const int* pz2 = &z;//pz2为const int* (底层const)
const int* const pz3= &z;//pz3为const int* const（同时包含底层和顶层const）

auto apz1 = pz1;//apz1为int*
auto apz2 = pz2;//apz2为const int*
auto apz3 = pz3;//apz3为cosnt int*
```

  

> 如果希望推导出的类型具有顶层const，则必须显式声明:

```cpp
int i = 0, &r = i;
const int ci = i, &cr = ci;
const auto f = ci; // deduced type of ci is int; f has type const int
const auto g = &ci; // deduced type of ci is const int*; g has type const int* const 
```

  

```cpp
int i = 0, &ri = i;
const int ci = 2, &rci = ci ;

const auto cb = i; //cb为const int型。因为规则3，cb被提升为const
const auto cb1 = ci; //同上

const auto ca1 = &i;//cal为常量指针。&i本是int*，因为规则3，强行将cal提升为常量指针int *const
const auto ccp = &ci;//本来&ci为const int *，因为规则3，加了const后，提示为const int * const
```

### 3.声明为auto引用：auto &

> 用auto声明指针类型时，用auto和auto*没有任何区别，但用auto声明引用类型时则必须加&

```cpp
int i = 0, &r = i;
const int ci = i, &cr = ci;
auto &g = ci; // g is a const int& that is bound to ci, Note: ci's reference has low-level const
auto &h = 42; // error: we can't bind a plain reference to a literal
const auto &j = 42; // ok: we can bind a const reference to a literal

```

  

```cpp
int i = 0, &ri = i;
const int ci = 2, &rci = ci ;
	
auto & j = i; //j为int &
auto & k = ci; // k为const int &
auto & h = 42; //错误,不能将非常量引用绑定字面值,这是引用&规则决定的

const auto &j2 = i; //j2为const int &，因为规则3，j2被提升为顶层const
const auto &k2 = ci; //k2为const int &
const auto &h2 = 42; //正确，可以为常量绑定字面值 

auto& m =  &i;//Error，无法从“int *”转换为“int *&” ,这是引用&规则决定的
auto& m1 = &ci;// Error，无法从“const int *”转换为“const int *&” ,这是引用&规则决定的
const auto &m2 = &i;//m2为int * const &
const auto &m3 = &ci;//m3为const int * const &

```

上例子中有3条是错误的，原因是：引用不能绑定表达式的计算结果，除非使用const。并且对于**普通指针而言，它提升到顶层const，只能为常量指针，而不能为指向常量的指针。**

### **4.从规则中总结错误用法**

（1）未初始化,auto 变量必须在定义时初始化，这类似于const关键字；

```text
auto a;
```

**未初始化的auto不能用**。

（2）**在同一行定义多个变量,定义在一个auto序列的变量必须始终推导成同一类型**

当在同一行声明多个变量时，这些变量必须是相同的类型，否则编译器将会报错，因为编译器实际只对第一个类型进行推导，然后用推导出来的类型定义其他变量。

```text
auto a = 1, b = 2;
auto c = 1, d = 1.11;

// 第二行中，c和d为不同类型。报错！
```

（3）auto不能推导的场景

**auto不能作为函数的参数和声明数组**

```cpp
void TestAuto(auto a) {

} // 报错
auto arr[3] = { 1,2,3 };//会报错

```

(4)**如果初始化表达式是引用，则去除引用语义**，如：

```cpp
int  a = 1;
int &b = a;

auto  c = b; // 此时c的类型被推导为 int32，而不是int32&
auto &c = b; // 此时c的类型才是int&

```

（5）**如果初始化表达式为const或volatile（或者两者兼有），则除去const/volatile语义。**

```cpp
const int a = 10;
auto  b = a;         // b的类型为int而非const int（去除const）
const auto c = a;   // 此时c的类型为const int

b = 100;   // 合法
c = 100;   // 非法

```

（6）**如果auto关键字带上&号，则不去除const语义**。

```cpp
const int a = 10;
auto &b = a;    // 因为auto带上&，故不去除const，b类型为const int

b = 10;   /非法

```

（7）**初始化表达式为数组时，auto关键字推导类型为指针**。

```cpp
int a[3] = { 1, 2, 3 };
auto b = a;

std::cout << typeid(b).name() << std::endl;   // 这里输出 int*

```

（8）如果表达式为数组且auto带上&，则推导类型为数组类型。

```cpp
int a[3] = { 1, 2, 3 };
auto &b = a;

std::cout << typeid(b).name() << std::endl; // 这里输出 int[3]

```

（9）函数或模板参数不能被声明为auto。

```cpp
void func(auto a)  // 错误
{
    //... 
}

template<auto T> // 错误
void deduceElement(T& element);

```

（10）auto不是一个真正的类型，仅仅是一个占位符，不能使用一些以类型为操作数的操作符，如sizeof或typeid：

```cpp
std::cout << sizeof(auto) << std::endl;        // 错误
std::cout << typeid(auto).name() << std::endl; // 错误

```

### 5.基于范围for循环配合auto

5.1

```cpp
for(auto x : vector)

```

auto会拷贝一份容器内的vector，在修改x时不会改变原容器当中的vector值，只会改变拷贝的vector。

5.2

```cpp
for(auto& x : vector)  // 非常量左值引用

```

当需要对原数据进行同步修改时，就需要添加&，即vector的引用。  
会在改变x的同时修改vector。

5.3

```cpp
for(auto&& x : vector)，非常量右值引用

```

当vector返回临时对象，使用auto&会编译错误，  
临时对象不能绑在non-const l-value reference （非常量左值引用），  
需使用auto&&（非常量右值引用），初始化右值时也可捕获

5.4

```cpp
for(const auto& x : vector) //，常量左值引用

```

使用 const 可以避免无意中修改数据的编程错误；  
使用 const 能让函数接收 const 和非 const 类型的实参，否则将只能接收非 const 类型的实参；  
使用 const 引用能够让函数正确生成并使用临时变量（当传入实参与形参符合自动转换要求时可以通过临时变量进行自动类型转换）。

当我们不希望拷贝原vector（拷贝需要申请新的空间），同时不愿意随意改变原vector，那么我们可以使用for(const auto& x : vector)，这样我们可以很方便的在不拷贝的情况下读取vector，同时不会修改vector。一般用在只读操作。

5.5

```cpp
const auto x : vector，常量左值引用

```

该操作相对于const auto& x : vector只是少了引用（&），即会申请新的空间（拷贝），不经常使用。  
const auto&& x：vector）,常量右值引用无实际意义，可以被常量左值引用替代

### **6、auto与auto&的区别**

使用auto标明一个变量，这个变量永远不会是引用变量。

使用auto&标明一个变量，这个变量有可能被编译器推导为引用变量。

区别1：改变原来的值

```text
auto a = obj.value();
a = 1001;

auto& b = obj.value();
b = 1001;
```

区别2：运行效率

```cpp
     void test()
    {
        vector<int> data(11111111,5);
        time_t t1, t2, t3;
        t1 = time(NULL);
        for (int i = 0;i < 100;i++)
        {
            auto testData = data;
        }
        t2 = time(NULL);
        for (int i = 0;i < 100;i++)
        {
            auto& testData = data;
        }
        t3 = time(NULL);
        cout << "auto消耗:" << difftime(t2, t1) << endl;
        cout << "auto&消耗:" << difftime(t3, t2) << endl;
    }

```

## 四、函数中的auto

```cpp
auto add = [v1 = 1.2, v2 = 2](int x, int y) -> double{
		return x + y + v1 + v2;
	};

auto sum = add(1, 2);

```

基本上，在函数中使用`auto`的情形大致可分为两类，在C++11中，`auto`被引入放到函数声明中返回类型的位置，用间接的方式来定义函数的返回类型，如下：

```cpp
//等价于 std::string someFunc(int i, double j);
 auto someFunc(int i, double j) -> std::string;

```

在C++14中，编译器可以直接推断出函数的返回类型了，所以可以写成下边的样子：

```cpp
auto someFunc(int i, double j) {
   //自动推断返回类型为std::string
   return std::to_string(i + j);
 }

```

## 五、auto&&

```cpp
void f(int &&a) {
    std::cout << "rvalue" << std::endl;
}

void f(int &a) {
    std::cout << "lvalue" << std::endl;
}

void g(int &&a) {
    std::cout << std::boolalpha;
    std::cout << "The type of a : " << type_to_string<decltype(a)>() << std::endl;
    std::cout << "The value type of a is lvalue : " << is_lvalue<decltype((a))> << std::endl;
    f(a);
}

g(10);
/////////

输出：

The type of a : int&&
The value type of a is lvalue : true
lvalue

```

## 合并两个重载函数变为一个函数模板，

```cpp
template<typename T>
void f(T&& a) {
    if constexpr(is_lvalue<decltype((a))>){
        std::cout << "lvalue" << std::endl;
    }else if constexpr(is_xvalue<decltype((a))>){
        std::cout << "xvalue" << std::endl;
    }
}
g(10);
////////
输出：

The type of a : int&&
The value type of a is lvalue : true
lvalue

```

  

```cpp
void g() {
    auto&& a = 10;
    std::cout << type_to_string<decltype(a)>() << std::endl;
    std::cout << type_to_string<decltype((a))>() << std::endl;
    if constexpr(is_lvalue<decltype((static_cast<decltype(a)>(a)))>){
        std::cout << "lvalue" << std::endl;
    }else if constexpr(is_xvalue<decltype((static_cast<decltype(a)>(a)))>){
        std::cout << "xvalue" << std::endl;
    }
}
输出

// a的类型
int&&
// a的值类型为lvalue
int&
// 经过完美转发后a的值类型重新变为了xvalue
xvalue
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbODIwNTE2ODkzLDkwNjA5NDgxMywtMTQwOD
ExMDc2NSwtNDAyMjMwNjY5LC05MDg2MDU5NzddfQ==
-->