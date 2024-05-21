# std::bind
可以预先**把某个可调用对象的某些参数绑定到已有的变量，产生一个新的可调用对象**。bind本身是一种**延迟计算的思想**，它本身可以绑定普通函数、全局函数、静态函数、类静态函数甚至是类成员函数。bind最终将会生成一个可调用对象，**这个对象可以直接赋值给std::function对象**，而std::bind绑定的可调用对象**可以是Lambda表达式或者类成员函数等可调用对象**。

使用方式：**bind（&要调用的函数，&对象， 要调用函数的参数1，要调用函数的参数2...，_1(bind函数的参数1)，_2(bind函数的参数2)...)**

```
#include <iostream>
#include <functional>
using namespace std;
 
int TestFunc(int a, char c, float f)
{
    cout << a << endl;
    cout << c << endl;
    cout << f << endl;
 
    return a;
}
 
int main()
{
    auto bindFunc1 = bind(TestFunc, placeholders::_1, 'A', 10.1); //表示绑定函数TestFunc的第二、三个参数分别为'A'和10.1，第一个参数由调用bindFunc1的第一个参数来指定
    bindFunc1(10);　　　　//输出为：10  A  10.1
 
    cout << "=================================\n";
 
    auto bindFunc2 = bind(TestFunc, std::placeholders::_2, placeholders::_1, 10.1);
    bindFunc2('B', 10);   //输出为：10  B  10.1
 
    cout << "=================================\n";
 
    auto bindFunc3 = bind(TestFunc, std::placeholders::_1, placeholders::_2, 10.1);
    bindFunc3(10, 'B');   //输出为：10  B  10.1
 
    return 0;
}
```

上面的auto关键字代表的是std::function<...>这一长串。

**bind能够在绑定时候就同时绑定一部分参数，未提供的参数则使用占位符表示，然后在运行时传入实际的参数值**。

> bind对于不事先绑定的参数，通过std::placeholders传递的参数是通过引用传递的；对于事先绑定的函数参数是通过值传

递的。

bind使用形式：

（1）bind（&f)()  假设f是一个全局函数，绑定全局函数并调用；

（2）bind (&A::f, A())()  假设A是一个构造函数为空的类，这个形式绑定了类的成员函数，故第二个参数需要传入一个成员

（成员静态函数除外）；

（3）bind (&A::f, _1)(new A()) 同上，效果是一样的，但是使用了占位符，使得没有固定的的对象，推荐。

注：使用的时候一定要注意指向的是没有this指针的函数（全局函数或静态成员函数），还是有this指针的函数。

后面一种必须要用bind()函数，而且要多一个参数，因为静态成员函数与非静态成员函数的参 数表不一样，原型相同的非静态函

数比静态成员函数多一个参数，即第一个参数this指针，指向所属的对象，任何非静态成员函数的第一个参数都是this指针。

std::function可以理解为函数指针。C++中把函数指针封装成了一个类,这也正是C++中无处不类的思想的体现,即

std::function,且是模板类。

1.保存普通函数

void printA(int a)
{ cout<<a<<endl; }
 
//传统的函数指针调用方式
typedef void(*funcPoint)(int);
funcPoint func(printA);
func(2);
 
//使用std::function的方式
std::function<void(int a)> func;
func = printA;
func(2);
2.保存成员函数

struct Foo {
    Foo(int num) : num_(num) {}
    void print_add1(int i) const { cout << num_+i << '\n'; }
    static void print_add2(int i)  {cout << i << '\n';}
    int num_;
};
 
//普通成员函数（第一种方法）
std::function<void(const Foo&, int)> f_add_display1 = &Foo::print_add1;
Foo foo(2);
f_add_display1(foo, 2);   //输出4
 
//普通成员函数（使用bind的方法）
std::function<void(int)> f_add_display2 = std::bind(&Foo::print_add1,foo,std::placeholders::_1);
f_add_display2(2);        //输出4
 
//静态成员函数
std::function<void(int)> f_add_display3 = &Foo::print_add2;
f_add_display3(2);        //输出2
 

3.保存lambda表达式

std::function<void()> func_1 = [](){cout<<"hello world"<<endl;};
func_1();
4.保存函数对象

class Functor
 {
 public:
     int operator()(const int &value)
     {
         std::cout << "函数对象\t:" << value << "\n";
         return value;
     }
 };
Functor fobj;
std::function<int(int)> func = fobj;
func(20);   //输出"函数对象 :20"
下面着重讲一些std::function在实际应用中要注意的问题。

1.静态成员函数情况

#include <iostream>  
#include <iomanip>  
#include <memory>  
#include <functional>  
  
typedef   std::function<void(int)>   HandlerEvent;
 
//定义一个成员变量
class Sharp{
   public:
       HandlerEvent   handlerEvent;   //为Sharp定义一个成员为std::function，相当于定义一个函数指针
};
//设置handlerEvent的值来动态装载事件响应函数
class Rectangle{
   private:
       std::string name;
       Sharp sharp;                   //它有成员为std::function
   public:
       void initial(void)
       const Sharp getSharp() const;
       static void onEvent(int param){std::cout<<"invode onEvent method, get parameter: "<<param<<std::endl;}
};
//类的实现方法
void Rectangle::initial(){
   sharp.handlerEvent = HandlerEvent(&Rectangle::onEvent);  //把一个成员函数赋给另一个成员对象sharp的成员std::function。
   std::cout<<"invode initial function!"<<std::endl;
}
const Sharp Rectangle::getSharp() const{
   return sharp;
}
//测试函数
int main(int argc, char *argv[]){
   std::cout<<"hi: "<<std::setw(50)<<"hello world!"<<std::endl;
   Rectangle rectangle;
   rectangle.initial();
   rectangle.getSharp().handlerEvent(23);        //从rectangle中访问某成员对象sharp并执行其std::function成员。等价于rectangle.onEvent(23)
   std::cin.get();
}
//输出结果
/*   hi:                                 hello world!
     invode initial function!
     invode onEvent method, get parameter: 23        */

注：这里使用了静态成员函数，如果把Rectangle前面的static去掉这段代码不能工作，编译都不能通过，因为静态成员函数与非静态成员函数的参数表不一样，原型相同的非静态函数比静态成员函数多一个参数，即第一个参数this指针，指向所属的对象，任何非静态成员函数的第一个参数都是this指 针，所以如果把Rectangle前面的static去掉，其函数原型等效于下面的一个全局函数：void onEvent(Rectangle* this, int);

2、非静态成员函数情况

将上例中Rectangle::onEvent(int param)前的static去掉改为非静态成员函数，则进行动态绑定使得程序正常运行，将Rectangle::initial(void)的定义修改为：

void Rectangle::initial(){
    //sharp.handlerEvent = HandlerEvent(&Rectangle::onEvent);
    sharp.handlerEvent = std::bind(&Rectangle::onEvent,this,std::placeholders::_1);//因onEvent函数需要一个参数，故使用一占位符
    std::cout << "invode initial function!" << std::endl;
}
这样，便动态装载函数成功。其它测试数据都不用进行修改。测试结果于上一样。

3、虚成员函数情况

定义类Square继承自Rectangle，将Rectangle::onEvent重载，定义一个新的Square::onEvent，Rectangle::initial中的函数不变，仍然使用Rectangle::onEvent进行绑定，则调用成员object.onEvent()时，具体执行Rectangle::onEvent还是Square::onEvent，看object所属对象的静态类型是Rectangle还是Square而定。

#include <iostream>  
#include <iomanip>  
#include <memory>  
#include <functional>  
  
typedef   std::function<void(int)>   HandlerEvent;  
  
//定义一个成员变量  
class Sharp  
{  
public:  
    HandlerEvent handlerEvent;   //定义一个成员为std::function，相当于函数指针
};  
//设置handlerEvent的值来动态装载事件响应函数  
class Rectangle  
{  
private:  
    std::string name;  
    Sharp sharp;  
public:  
    void initial(void);  
    const Sharp getSharp() const;  
    virtual void onEvent(int param)  
    {   
        std::cout << "invode Rectangle's onEvent method,get parameter: " << param << std::endl;  
    }  
};  
//Square类来继承Rectangle类，并重写onEvent  
class Square : public Rectangle{  
public:  
    void onEvent(int param)  
    {  
        std::cout << "invode Square's onEvent method,get parameter: " << param << std::endl;  
    }  
};  
  
//类的实现方法  
void Rectangle::initial()  
{  
    sharp.handlerEvent = std::bind(&Rectangle::onEvent,this,std::placeholders::_1);   //把当前类的一个成员函数赋给成员sharp的成员std::function
    std::cout << "invode initial function!" << std::endl;  
}  
const Sharp Rectangle::getSharp() const  
{  
    return sharp;  
}  
  
//测试函数  
int main(int argc, char *argv[]){  
    std::cout << "hi: " << std::setw(50) << "hello world!" << std::endl;  
  
    Rectangle rectangle;  
    rectangle.initial();   
    rectangle.getSharp().handlerEvent(23);    
  
    Square square;  
    square.initial();  
    square.getSharp().handlerEvent(33);  
  
    std::cin.get();  
}  
//输出结果  
/*  hi:                                       hello world! 
    invode initial function! 
    invode Rectangle's onEvent method,get parameter: 23 
    invode initial function! 
    invode Square's onEvent method,get parameter: 33    */  
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjE0MDk2MTMyMywtMTU2OTg5OTgxM119
-->