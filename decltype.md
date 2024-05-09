## 概念

decltype是 C++11 新增的一个关键字，它和 auto 的功能一样，都用来**在编译时期进行自动类型推导**。
decltype 是“declare type”的缩写，译为“声明类型”。 **decltype关键字主要作用是选择并返回操作数的数据类型**，在此过程中编译器分析表达式并得到它的类型，却**不实际计算表达式的值**。 例如 decltype(fun()) sum=x;，编译器实际上没有调用fun函数，而是使用fun()的返回值类型作为sum的数据类型。 下面我们就来详细学习一下它的用法以及如何根据表达式推导类型。

## 基础用法

### 语法
```cpp
auto varname = value; 
decltype(exp) varname [= value]; 
// varname 表示变量名，value 表示赋给变量的值
// exp 表示一个表达式，方括号[ ]表示可有可无。` 
```
decltype 和auto的区别：
-   auto 必须要根据=右边的初始值 value 推导出变量的类型；所以**auto要求变量必须初始化**，也就是在定义变量的同时必须给它赋值，不然会编译报错。
-   decltype 根据 exp 表达式推导出变量的类型，跟=右边的 value 没有关系。 所以**decltype 不强制要求初始化**。

```cpp

#include <iostream>
int main() {
 auto x = 5; //必须初始化，不然会报错，auto是根据初始值推导出变量的类型的
 decltype(10)y; //可以不初始化，因为是根据()里面的内容推导出来的类型
 decltype(10)z = 100; //也可以不初始化。
 auto str1 = "decltype_test";
 decltype(str1) str2 = "decltype_test2";
}
```

### cv 限定符

decltype处理顶层const和引用的方式与auto有些不同，如果decltype使用的表达式是个变量，则decltype返回该变量的类型(包括顶层const和引用在内)。例如：

```cpp

#include <iostream>
int main(){
 const int ci=0, &cj=ci;
 decltype(ci) x=0; //x的类型是const int；
 decltype(cj) y=x; //y的类型是const int &，y绑定到变量x；
 decltype(cj) z;   //错误，z是引用，必须初始化；
}
```

对于非引用(指针)，decltype关键字和auto关键字的作用类似，但是**保留表达式的引用及const限定符**。
decltype能够精确地推导出表达式本身的类型，不会像auto那样在某些情况下舍弃掉引用和cv限定符。

### 引用与指针

使用关键字decltype的时候，左值和右值有所不同。
如果表达式的求值结果是**左值**，decltype作用于该表达式(不是变量)**得到引用**。例如，假定p的类型是`int*`，因为解引用符(*)生成左值，所以decltype(*p)的结果是int&。
如果表达式的求值结果是**右值**，得到**指向指针的指针**，例如，decltype(&p)的结果是int*，因为取地址运算符生成右值。

-   **decltype与指针**：decltype处理指针时需要**保留指针**，这点和auto是有区别。例如：

```cpp

#include <iostream>
int main(){
 int tempA = 2;
 int *ptrTempA = &tempA;
 //1.常规使用dclTempA为一个int *的指针；
 decltype(ptrTempA) dclTempA;
 //2.需要特别注意，表达式内容为解引用操作，
 //dclTempB为一个引用int &，引用必须初始化；
 // 否则错误：error: 'dclTempB' declared as reference but not initialized
 decltype(*ptrTempA) dclTempB = tempA;
 // &ptrTempA 是指向指针的指针， 是个右值， 所以类型是int*
 decltype(&ptrTempA) dclTempC = ptrTempA
}
```

-   **decltype与引用**：decltype处理引用时需要**保留引用**，这点和auto是有区别。例如：
```cpp
#include <iostream>
int main(){
 //非const引用
 int tempA = 0, &refTempA = tempA;
 //1.dclTempA为引用，绑定到tempA；
 decltype(refTempA) dclTempA = tempA;
 //2.双层括号表示引用，dclTempB为引用，绑定到tempA；
 decltype((tempA)) dclTempB = tempA;
 //const引用；
 const int ctempA = 1, &crefTempA = ctempA;
 //1.dclTempE为const用，可以绑定到非const变量；
 decltype(crefTempA) dclTempE = tempA;
 //2.dclTempF为const引用，可以绑定到const；
 decltype(crefTempA) dclTempF = ctempA;
 //3.dclTempG为const引用，绑定到一个字面值；
 decltype(crefTempA) dclTempG = 0;
}
```

### 模板声明

在C++11中，decltype最主要的用处可能就是用来声明一个函数模板，在这个函数模板中返回值的类型取决于参数的类型。举个例子，假设我们想写一个函数，这个函数中接受一个支持方括号索引（也就是"[]"）的容器作为参数，验证用户的合法性后返回索引结果。这个函数的返回值类型应该和索引操作的返回值类型是一样的。

c++

复制代码

`#include <iostream>
#include <vector>
template<typename Container, typename Index>
auto f(Container& c, Index i) -> decltype(c[i]) {
 return c[i];
}
int main() {
 std::vector<int> v(8);
 f(v, 3);   // 1
 f(v, 3) = 5;  // 2
}` 

到了 C++14，我们这样写也是对的：

c++

复制代码

`template<typename Container, typename Index>
auto f(Container& c, Index i) {  // C++ 14
 return c[i];
} // 返回了 int` 

然而，这还是有问题。假设 c 中对象的类型为 int，则 c[i] 返回的类型为 int&，经过 auto 后，引用会被忽略，变为 int。这时，返回的就是右值而不是左值。上面的语句②就会编译错误。如果想要返回左值，则必须这样写：

c++

复制代码

`template<typename Container, typename Index>
decltype(auto) f(Container& c, Index i) {  // C++ 14
 return c[i];
} // 返回了 int&` 

这个容器是通过非`const`左值引用传入的，因为通过返回一个容器元素的引用是来修改容器是被允许的。但是这也意味着不可能将右值传入这个函数。右值不能和一个左值引用绑定（除非是`const`的左值引用，但这里不是这种情况）。

当然，如果想传递一个`右值容器`给f模板函数，那么`右值容器`作为一个临时对象，在f函数结束时被销毁，如果返回这个临时容器中元素的引用，很显然这个引用在函数结束时也被悬空。但是有时可能用户也仅仅是想拷贝这个临时容器的一个元素，那么有没有办法做到？当然可以，看下面示例：

c++

复制代码

`template<typename Container, typename Index>
auto f(Container&& c, Index i) -> decltype(std::forward<Container>(c)[i]) // C++11
{
 return std::forward<Container>(c)[i];
}` 

首先，Container参数用了万能引用，可以接收任何类型，左值或右值都可以。我们用std::forward实现我们想要的效果。调用forward，若原来是一个右值，那么他转出来就是一个右值，否则为一个左值。

c++

复制代码

`#include <iostream>
#include <vector>
template<typename Container, typename Index>
auto f(Container&& c, Index i) -> decltype(std::forward<Container>(c)[i]) // C++11
{
 return std::forward<Container>(c)[i];
}
int main() {
 std::vector<int> v(8);
 f(v, 3);   // 1
 f(v, 3) = 5;  // 2
 std::cout << v[3] << "\n";  // output 5
 auto x = f((std::vector<int>){1,2,3,4,5,6,7}, 3);  // 3
 std::cout << x << "\n";  // output 4
}` 

### 非静态成员的类型

我们都知道 auto 并不适用于所有的自动类型推导场景，auto 只能用于类的静态成员，不能用于类的非静态成员（普通成员）。在某些特殊情况下 auto 用起来非常不方便，甚至压根无法使用，所以 decltype 关键字也被引入到 C++11 中。如果我们想推导非静态成员的类型，这个时候就必须使用 decltype 了。看下面例子：

cpp

复制代码

`#include <vector>
using namespace std;
template <typename T>
class Base {
public:
 void func(T& container) {
 m_it = container.begin();
 }
private:
 //T::iterator并不能包括所有的迭代器类型，
 //当 T 是一个 const 容器时，应当使用 const_iterator。
 // typename T::iterator m_it;  //注意这里 
 decltype(T().begin()) m_it;  //注意这里
};
int main() {
 const vector<int> v;		//注意这里
 Base<const vector<int>> obj;
 obj.func(v);
 return 0;
}` 

## decltype 推导规则

使用 `decltype(exp)` 获取类型时，编译器将根据以下三条规则得出结果：

-   如果 exp 是一个不被括号()包围的表达式，或者是一个类成员访问表达式，或者是一个单独的变量，那么 decltype(exp) 的类型就和 exp 一致，这是最普遍最常见的情况。 注：这里的`不被括号()包围`是指表达式exp自身带的小括号，而不是decltype(exp)中的小括号。
-   如果 exp 是函数调用，那么 decltype(exp) 的类型就和函数返回值的类型一致。
-   如果 exp 是一个左值，或者被括号( )包围，那么 decltype(exp) 的类型就是 exp 的引用；假设 exp 的类型为 T，那么 decltype(exp) 的类型就是 T&。
-   如果 exp 是T类型的x值，那么decltype(exp )的结果类型为T&&。 注：x值（xvalue）是C++11新引入的值的种类，介于传统的左值和右值之间。最常见的x值为无名右值引用。

cpp

复制代码

`#include <iostream>
using namespace std;
class Base{
public:
 int i;
 static int si;
 string str;
 float ff;
};
int Base::si = 0;
//函数声明
int& func_int_r(int, char);  //返回值为 int&
int&& func_int_rr(void);  //返回值为 int&&
int func_int(double);  //返回值为 int
const int& fun_cint_r(int, int, int);  //返回值为 const int&
const int&& func_cint_rr(void);  //返回值为 const int&&
int main(){
 { // 规则一
 int n = 0;
 const int &r = n;
 Base base = Base();
 decltype(n) a = n;  //n 为 int 类型，a 被推导为 int 类型
 decltype(r) b = n;     //r 为 const int& 类型, b 被推导为 const int& 类型
 decltype(Base::si) c = 0;  //si 为类 Base 的一个 int 类型的成员变量，c 被推导为 int 类型
 //str 为类 Base 的一个 string 类型的成员变量， url 被推导为 string 类型
 decltype(base.str) url = "http://www.baidu.com"; 
 }
 { // 规则二
 //decltype类型推导
 int n = 100;
 decltype(func_int_r(100, 'A')) a = n;  //a 的类型为 int&
 decltype(func_int_rr()) b = 0;  //b 的类型为 int&&
 decltype(func_int(10.5)) c = 0;   //c 的类型为 int
 decltype(fun_cint_r(1,2,3))  x = n;    //x 的类型为 const int &
 decltype(func_cint_rr()) y = 0;  // y 的类型为 const int&&
 }
 { // 规则三
 const Base obj = Base();
 //带有括号的表达式
 decltype(obj.i) a = 0;  //obj.i 为类的成员访问表达式，符合推导规则一，a 的类型为 int
 decltype((obj.i)) b = a;  //obj.i 带有括号，符合推导规则三，b 的类型为 int&。
 //加法表达式
 int n = 0, m = 0;
 decltype(n + m) c = 0;  //n+m 得到一个右值，符合推导规则一，所以推导结果为 int
 decltype(n = n + m) d = c;  //n=n+m 得到一个左值，符号推导规则三，所以推导结果为 int& 
 }
 { // 规则四
 int x = 0;
 decltype(std::move(x)) a = 1;  //std::move(x)得到一个右值引用，符合推导规则四，所以推导结果为int&&
 }
 return 0;
}` 

## decltype(auto) - C++14

decltype(auto)是C++14新增的类型指示符，可以用来声明变量以及指示函数返回类型。

-   当decltype(auto)被用于声明变量时，该变量必须立即初始化。假设该变量的初始化表达式为e，那么该变量的类型将被推导为decltype(e)。也就是说在推导变量类型时，先用初始化表达式替换decltype(auto)当中的auto，然后再根据decltype的语法规则来确定变量的类型。
-   当decltype(auto)被用于指示函数的返回值类型时。假设函数返回表达式e，那么该函数的返回值类型将被推导为decltype(e)。也就是说在推导函数返回值类型时，先用返回值表达式替换decltype(auto)当中的auto，然后再根据decltype的语法规则来确定函数返回值的类型。

c++

复制代码

`#include <iostream>
  
template<typename T> 
T compileTypeId();  // 用于编译出错，根据错误信息打印出T的类型
  
struct Base {int x;};
  
int a = 0;
Base base;
decltype(auto) g1() {return base.x;}
decltype(auto) g2() {return std::move(a);}
decltype(auto) g3() {return (a);}
decltype(auto) g4() {return (0);}
  
int main() {
 decltype(auto) i1 = a;
 decltype(auto) i2 = std::move(a);
 decltype(auto) i3 = (base.x);
 decltype(auto) i4 = (0);
 compileTypeId<decltype(i1)>();  // int
 compileTypeId<decltype(i2)>();  // int&&
 compileTypeId<decltype(i3)>();  // int&
 compileTypeId<decltype(i4)>();  // int
 compileTypeId<decltype(g1())>();  // int
 compileTypeId<decltype(g2())>();  // int&&
 compileTypeId<decltype(g3())>();  // int&
 compileTypeId<decltype(g4())>();  // int
}` 

根据编译错误信息可以看出实际的类型：

csharp

复制代码

``<source>:21: undefined reference to `int compileTypeId<int>()'
<source>:22: undefined reference to `int&& compileTypeId<int&&>()'
<source>:23: undefined reference to `int& compileTypeId<int&>()'
<source>:24: undefined reference to `int compileTypeId<int>()'
<source>:25: undefined reference to `int compileTypeId<int>()'
<source>:26: undefined reference to `int&& compileTypeId<int&&>()'
<source>:27: undefined reference to `int& compileTypeId<int&>()'
<source>:28: undefined reference to `int compileTypeId<int>()'`` 

## 总结

auto 虽然在书写格式上比 decltype 简单，但是它的推导规则复杂，有时候会改变表达式的原始类型；而 decltype 比较纯粹，它一般会坚持保留原始表达式的任何类型，让推导的结果更加原汁原味。

`decltype`几乎总是得到一个变量或表达式的类型而不需要任何修改，对于非变量名的类型为`T`的左值表达式，`decltype`总是返回`T&`。

通常，我们使用 auto类型说明符和 decltype 类型说明符来帮助编写模板库，使用 auto 并 decltype 声明其返回类型的模板函数取决于其模板参数的类型。

好了，到这里，decltype基本就介绍完了。

  

作者：Codemaxi  
链接：https://juejin.cn/post/7092020261972606983  
来源：稀土掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwOTI0MzkzMTAsNDM5MzY4NDkyLC0zMj
I5NzU3NjQsLTYzOTI3NTA4LC0zMjk3NjA2Niw5NjIxOTY3NTQs
LTE4MzQ2NjcwMzEsLTY5NTA1MDE2NV19
-->