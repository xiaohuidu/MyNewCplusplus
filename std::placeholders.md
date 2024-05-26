### std::bind
1.摘自《Effective Modern C++》中一段对std::bind描述的话
```
std::bind是C++11对C++98中对std::bind1st和std::bind2nd的继承,但是,非正式地说,
它自2005年以来一直是标准库的一部分.标准化委员会通过了一份称为TR1的文件,其中包
括bind的具体实现(在TR1中,bind位于不同
的命名空间中,因此它是std::tr1::bind而不是std::bind,有几个接口细节是不一样的)
这段历史意味着一些程序员有十年或更长时间的使用绑定.如果你是其中之一,你可能不
愿意放弃一个工具,这是可以理解的,但在这种情况下,变化是好的,因为在在C++11中,lambdas
几乎总是比std::bind更好的选择.lambdas(兰布达斯)的理由不仅仅是更有力,而是无懈可击。
```


### 2.std::placeholders
定义如下：
```
namespace placeholders {
  extern /* unspecified */ _1;
  extern /* unspecified */ _2;
  extern /* unspecified */ _3;
  // ...
}
 ```
 
这个命名空间声明了未指定数量的对象：_1、_2、_3、...，这些对象用于在函数bind的调用中指定占
位符. 

当调用bind返回的函数对象时,带有占位符_1的参数为被调用中的第一个参数替换，_2被被调用中的第一个参数替换,以此类推.
 ```cpp
using namespace std::placeholders;
auto bound_fn = std::bind (fn,100,_1);
bound_fn(5);  // calls fn(100,5), i.e.: replacing _1 by the first argument: 5 
 ```
 
 这些占位符对象的类型未指定(这取决于库实现,见is_placeholder),但在所有情况下,它们的类型至少
应该是无抛出默认可构造和无抛出复制可构造.不管分配操作或其他构造函数是否真正，任何复制赋值
或移动构造函数也不能抛出。
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU5Nzk4NTIxNCwtMjMxNjc4NjU1LC0xOT
IzOTQxMjMyLDczMDk5ODExNl19
-->