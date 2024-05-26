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

### 3.std::bind&std::placeholders
3.0 std::bind的使用场景，头文件及原型
简介：
std::bind是这样一种机制，它可以预先把指定可调用实体的某些参数绑定到已有的变量，产生一个新
的可调用实体。可将std::bind函数看作一个通用的函数适配器，它接受一个可调用对象，生成一个新
的可调用对象来“适应”原对象的参数列表。std::bind将可调用对象与其参数一起进行绑定，绑定后的
结果可以使用std::function保存。std::bind主要有以下几个作用：
 
将可调用对象和其参数绑定成一个仿函数
只绑定部分参数，减少可调用对象传入的参数
改变参数绑定顺序
 
 
使用场景：
先将可调用的对象保存起来，在需要的时候再调用，是一种延迟计算的思想。不论是普通函数、函数对
象、还是成员函数，成员变量都可以绑定，其中成员函数都可以绑定是相当灵活的。
 
头文件：
#include <functional>
 
原型：
(1)
template< class F, class... Args >
/*unspecified*/ bind( F&& f, Args&&... args );
 
(2)
template< class R, class F, class... Args >
/*unspecified*/ bind( F&& f, Args&&... args );

3.1 std::bind单独使用，绑定普通函数
#include <functional>
#include <iostream>
using namespace std;
double myDivide(double x, double y)  
{
        return x/y;
}
int main(){
        // bind的第一个参数是函数名，普通函数做实参时，会隐式转换成函数指针,因此std::bind (myDivide,1,2)等价于std::bind (&myDivide,1,2)
        auto fn_half = std::bind(myDivide, 1, 2);  
        std::cout << fn_half() << std::endl;                        // 0.5
}


3.2 std::bind单独使用，绑定类的成员函数
#include <functional>
#include <iostream>
using namespace std;
struct Foo {
    void print_sum(int n1, int n2) 
    {   
        std::cout << n1+n2 << '\n';
    }   
    int data = 10; 
};
int main() 
{
    Foo foo;
    /*  
      bind绑定类成员函数时，第一个参数表示对象的成员函数的指针，第二个参数表示对象的地址。
      必须显示的指定&Foo::print_sum，因为编译器不会将对象的成员函数隐式转换成函数指针，
      所以必须在Foo::print_sum前添加&。使用对象成员函数的指针时，必须要知道该指针属于哪个 
      对象,因此第二个参数为对象的地址 &foo；
  */
    auto f = std::bind(&Foo::print_sum, &foo, 95, 5); 
    f(); // 100
}
 
 



3.3 std::bind单独使用，绑定一个引用参数
#include <iostream>
#include <functional>
#include <vector>
#include <algorithm>
#include <sstream>
using namespace std::placeholders;
using namespace std;
 
ostream & print(ostream &os, const string& s, char c)
{
        os << s << c;
        return os; 
}
 
int main()
{
        vector<string> words{"hello", "world", "this", "is", "C++11"};
        ostringstream os; 
        char c = ' ';
        for_each(words.begin(), words.end(), 
                        [&os, c](const string & s){os << s << c;} );
        cout << os.str() << endl;
 
        ostringstream os1;
        // ostream不能拷贝，若希望传递给bind一个对象，
        // 而不拷贝它，就必须使用标准库提供的ref函数
        for_each(words.begin(), words.end(),
                        bind(print, ref(os1), _1, c));
        cout << os1.str() << endl;
}



3.4 使用std::bind和std::placeholders混用达到减少参数使用和调整参数次序的效果
#include <functional>
#include <iostream>
using namespace std;
 
double myDivide(double x, double y)  
{
        return x/y;
}
int main(){
        // 减少一个参数
        auto fn_half_1 = std::bind(myDivide, std::placeholders::_1, 2);  
        std::cout << fn_half_1(1) << std::endl; // 0.5                         
 
        // 减少两个参数
        auto fn_half_2 = std::bind(myDivide, 1, 2);  
        std::cout << fn_half_2() << std::endl;  // 0.5                         
 
        // 将调用的第一个参数和第二个参数互换
        auto fn_half_3 = std::bind(myDivide, std::placeholders::_2, std::placeholders::_1);  
        std::cout << fn_half_3(1, 2) << std::endl; // 2
} 



3.5 使用std::bind和std::placeholders达到调整参数次序的效果的另一个例子
#include <functional>
#include <iostream>
int TestFunc(int a, char c, float f)
{
        std::cout << a << std::endl;
        std::cout << c << std::endl;
        std::cout << f << std::endl;
        return a;
}
// 第一个参数和函数第一个参数匹配（int）,第二个参数和第二个参数匹配（char）,第三个参数和第三个参数匹配
auto fun1 = std::bind(TestFunc, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3);
 
// 通过占位符调整顺序,第二个参数和函数第一个参数匹配（int）,第三个参数和第二个参数匹配（char）,第一个参数和第三个参数匹配
auto fun2 = std::bind(TestFunc, std::placeholders::_2, std::placeholders::_3, std::placeholders::_1);
 
//  第一个参数和函数第一个参数匹配（int）,第二个参数和第二个参数匹配（char）,第三个参数默认为98.77, 如果第三个参数也要填的话，会被忽略掉
auto fun3 = std::bind(TestFunc, std::placeholders::_1, std::placeholders::_2, 98.77);
int main(){
        fun1(30, 'C',100.1); 
        fun2(100.1, 30, 'C');
        fun3(30,'C');
        fun3(30,'C',9.8); // 9.8 被忽略
        return 0;
}
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMzY1NzUwMywtMjMxNjc4NjU1LC0xOT
IzOTQxMjMyLDczMDk5ODExNl19
-->