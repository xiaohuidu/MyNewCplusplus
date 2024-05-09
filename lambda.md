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
```
auto f = [] (int a, int b) -> int
{
        return a + b + 42;
};
003A3BC0 6A 03                push        3  
003A3BC2 6A 04                push        4  
003A3BC4 8D 4D EB             lea         ecx,[f]  
003A3BC7 E8 04 E4 FF FF       call        <lambda_f2fe7ac06244f603e089b2eaef4ffd5c>::operator() (03A1FD0h) 
cout << f(4, 3) << endl;
```

可以看到，调用该functor时，call到的是一个lambda对象的operator()位置，该位置反汇编代码如下：
```
00BA1FD0 55                   push        ebp  
00BA1FD1 8B EC                mov         ebp,esp  
00BA1FD3 81 EC CC 00 00 00    sub         esp,0CCh  
; 省略一堆东西
00BA1FF0 89 4D F8             mov         dword ptr [this],ecx  
    49:         return a + b + 42;
00BA1FF3 8B 45 0C             mov         eax,dword ptr [b]  
00BA1FF6 8B 4D 08             mov         ecx,dword ptr [a]  
00BA1FF9 8D 44 01 2A          lea         eax,[ecx+eax+2Ah]  
    50:     };
; 省略一堆东西
00BA2000 8B E5                mov         esp,ebp  
00BA2002 5D                   pop         ebp  
00BA2003 C2 08 00             ret         8  
```
注意：

1.  如果在lambda表达式中忽略括号和形参列表，则相当于指定的函数没有入参。
2.  lambda表达式中不能指定参数的默认值。
3.  如果忽略返回值类型，则由编译器做自动类型推断，详细规则如第一部分所示。

**捕获列表**
由于lambda表达式是在某函数内定义的，因此我们可能希望其能使用**函数内的局部变量**，这时则可以使用所谓捕获列表。总体说来，一共有三种捕获方式：**值捕获**、**引用捕获**和**外部捕获**。

**值捕获**
值捕获和参数传递中的值传递类似，被捕获的变量的值在Lambda表达式创建时通过值拷贝的方式传入，因此随后对该变量的修改不会影响影响Lambda表达式中的值。在VS2017中创建如下代码并加以分析：
```
int testFunc1()
{
    int nTest1 = 23;

    auto f = [nTest1] (int a, int b) -> int
    {
        return a + b + 42 + nTest1;
        //nTest1 = 333;              不能在lambda表达式中修改捕获变量的值
    };

    cout << f(4, 3) << "&nTest1=" << nTest1 << endl;
}
```

需要注意的是，不能在lambda表达式中修改捕获变量的值。其实根据上面的反汇编分析，我们已经知道，lambda表达式中的代码是在一个单独的函数中执行的，这个函数在调用时创建了自己的栈帧，而其使用的nTest1局部变量在testFunc1的栈帧中，虽然通过ebp可以进行栈帧回溯，但显然这是一种不合情理的做法。因此可以断定，捕获列表中出现的局部变量一定会通过某种方式传递给lambda匿名类。到底采用的是哪种方式呢？我们来揭晓答案：

```
    49:     auto f = [nTest1] (int a, int b) -> int
    50:     {
    51:         return a + b + 42 + nTest1;
    52:     };
01183C08 8D 45 E8             lea         eax,[nTest1]  
01183C0B 50                   push        eax  
01183C0C 8D 4D DC             lea         ecx,[f]  
01183C0F E8 0C E3 FF FF       call        <lambda_ed51e51ff76776313a28b716c94bbc2d>::<lambda_ed51e51ff76776313a28b716c94bbc2d> (01181F20h)  
    ; 省略无关代码
    54:     cout << f(4, 3) << "&nTest1=" << nTest1 << endl;
01183C1D 8B 45 E8             mov         eax,dword ptr [nTest1]  
01183C20 50                   push        eax  
01183C21 68 40 BE 18 01       push        offset string "&nTest1=" (0118BE40h)  
01183C26 6A 03                push        3  
01183C28 6A 04                push        4  
01183C2A 8D 4D DC             lea         ecx,[f]  
01183C2D E8 4E F7 FF FF       call        <lambda_ed51e51ff76776313a28b716c94bbc2d>::operator() (01183380h)  
```
基本可以断定，nTest是在lambda匿名类构造时传入的。并且传入的是nTest1的引用（由于C++的引用本身就是语法糖，反汇编层面看到的是指针，但是结合源码分析不难得出这里应该是引用）。下面跟踪进其构造函数一探究竟：
```
01181F20 55                   push        ebp  
01181F21 8B EC                mov         ebp,esp  
01181F23 81 EC CC 00 00 00    sub         esp,0CCh  
; 省略部分现场保护和堆栈填充int 3的代码
01181F40 89 4D F8             mov         dword ptr [this],ecx 
; 01181F40 89 4D F8             mov         dword ptr [ebp-8],ecx 
01181F43 8B 45 F8             mov         eax,dword ptr [this]  
; 01181F43 8B 45 F8             mov         eax,dword ptr [ebp-8]  
01181F46 8B 4D 08             mov         ecx,dword ptr [<nTest1>] 
; 01181F46 8B 4D 08             mov         ecx,dword ptr [ebp+8] 
01181F49 8B 11                mov         edx,dword ptr [ecx]  
01181F4B 89 10                mov         dword ptr [eax],edx  
01181F4D 8B 45 F8             mov         eax,dword ptr [this] 
; 01181F4D 8B 45 F8             mov         eax,dword ptr [ebp-8]
; 省略部分现场恢复代码
01181F53 8B E5                mov         esp,ebp  
01181F55 5D                   pop         ebp  
01181F56 C2 04 00             ret         4  
```

由此可以确定，lambda匿名类会将捕获参数中的变量添加到其成员变量中，并设置一个带有该参数引用类型的构造函数。最后再来看看在其operator()函数中是怎样使用捕获变量的：
```
    49:     auto f = [nTest1] (int a, int b) -> int
    50:     {
01183380 55                   push        ebp  
01183381 8B EC                mov         ebp,esp  
01183383 81 EC CC 00 00 00    sub         esp,0CCh  
; 省略部分现场保护和堆栈填充int 3的代码
011833A0 89 4D F8             mov         dword ptr [this],ecx  
    51:         return a + b + 42 + nTest1;
011833A3 8B 45 08             mov         eax,dword ptr [a]  
011833A6 03 45 0C             add         eax,dword ptr [b]  
011833A9 8B 4D F8             mov         ecx,dword ptr [this]  
011833AC 8B 11                mov         edx,dword ptr [ecx]  
011833AE 8D 44 10 2A          lea         eax,[eax+edx+2Ah]  
    52:     };
; 省略部分现场恢复代码
011833B5 8B E5                mov         esp,ebp  
011833B7 5D                   pop         ebp  
011833B8 C2 08 00             ret         8  
```
真相大白了，值捕获时，C++编译器在构建lambda表达式的匿名类时将局部变量的引用传入，并在构造函数中完成对相应成员变量的赋值。在调用其operator()函数时，如果用到了捕获列表中的局部变量，则从给匿名类对象的成员变量中取出。

### 引用捕获

使用引用捕获一个外部变量，需在捕获列表变量前面加上一个引用说明符&。如下：
```
void fnTest()
{
    int nTest1 = 23;

    auto f = [&nTest1] (int a, int b) -> int
    {
        cout << "In functor before change nTest=" << nTest1 << endl;    //nTest1=23333
        nTest1 = 131;
        cout << "In functor after change nTest=" << nTest1 << endl;     // nTest1 = 131
        return a + b + 42 + nTest1;
    };

    nTest1 = 23333;     

    cout << f(4, 3) << "&nTest1=" << nTest1 << endl;        //nTest1 = 23333
}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTU0OTAwNjM0MywtMTM2MDgxNzEwOSwzOD
U4MDk2MjIsOTQ2NTM0NzUzLDE1OTE0MzQwMzddfQ==
-->