介绍

**packaged_task**类模板也是定义于future头文件中，它包装任何**可调用 (Callable) 目标**，包括**函数、 lambda 表达式、 bind 表达式或其他函数对象**，使得能异步调用它，其返回值或所抛异常被存储于能通过 std::future 对象访问的共享状态中。简言之，**将一个普通的可调用函数对象转换为异步执行的任务**。模板声明如下：
```cpp
template< class R, class ...Args > 
class packaged_task< fn(Args...)>;
```

其中：
fn 是可以调用目标
Args 函数入参

通过packaged_task包装后，可以通过thread启动或者仿函数形式启动，其执行结果返回值或所抛异常被存储于能通过 std::future 对象访问的共享状态中。

实例

在packaged_task成员中，进行了()运算符重载，因此我们可以直接调用这个开始启动任务。

void operator()( ArgTypes... args );
以 args 为参数调用存储的任务。
1
2
实例代码如下：

#include <iostream>           // std::cout
#include <thread>             // std::thread
#include <chrono>
#include <future>

using namespace std;
//普通函数
int Add(int x, int y)
{
    return x + y;
}


void task_lambda()
{
    //包装可调用目标时lambda
    packaged_task<int(int,int)> task([](int a, int b){ return a + b;});
    
    //仿函数形式，启动任务
    task(2, 10);
    
    //获取共享状态中的值,直到ready才能返回结果或者异常
    future<int> result = task.get_future();
    cout << "task_lambda :" << result.get() << "\n";
}

void task_thread()
{
    //包装普通函数
    std::packaged_task<int (int,int)> task(Add);
    future<int> result = task.get_future();
    //启动任务，非异步
    task(4,8);
    cout << "task_thread :" << result.get() << "\n";
        
    //重置共享状态
    task.reset();
    result = task.get_future();

    //通过线程启动任务，异步启动
    thread td(move(task), 2, 10);
    td.join();
    //获取执行结果
    cout << "task_thread :" << result.get() << "\n";
}

int main(int argc, char *argv[])
{
    task_lambda();
    task_thread();

    return 0;
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
运行结果：

task_lambda :12
task_thread :12
task_thread :12
1
2
3
区别

那么std::async 、std::promise、std::packaged_task以及std::future之间有什么联系和区别呢？

通过 std::async 和 std::promise 以及以上内容可以看出：

std::future是用于获取将来共享状态的运行结果或异常，相当于一个中间件，std::async 、std::promise、std::packaged_task都离不开它的帮助；

std::packaged_task用于包装可调用目标，以便异步执行任务；

std::promise用于设置共享状态的值，可以用于线程间交流，这个是比较特殊的。

std::async是最优雅地方式启动任务异步执行；在多数情况下，建议使用asyn开启异步任务，而不是使用packaged_task方式。
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjExNzAzNTcxMCwtMjQ4ODU2ODYyXX0=
-->