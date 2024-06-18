C++11提供了日期时间相关的库chrono，通过chrono库可以很方便的处理日期和时间, chrono是一个time library, 源于boost，现在已经是C++标准。

要使用chrono库，需要#include<chrono>，其所有实现均在std::chrono namespace下。注意标准库里面的每个命名空间代表了一个独立的概念。所以下文中的概念均以命名空间的名字表示！ chrono是一个**模版库**，使用简单，功能强大，只需要理解**三个概念：duration、time_point、clock**

 **1. Durations**

std::chrono::duration 表示一段时间，比如两个小时，12.88秒，半个时辰，一炷香的时间等等，只要能换算成秒即可。
```cpp
template <class Rep, class Period = ratio<1> > class duration;

std::chrono::duration<int,ratio<60,1>> 
表示持续的一段时间,这段时间的单位是由ratio<60,1>决定的,int表示这段时间的值的类型,函数返回的类型还是一个时间段duration
```

其中

Rep表示一种数值类型，用来表示Period的数量，比如int float double

Period是ratio类型，用来表示【用秒表示的时间单位】比如second milisecond

常用的duration<Rep,Period>已经定义好了，在std::chrono::duration下：
```cpp
ratio<3600, 1>  hours

ratio<60, 1>  minutes

ratio<1, 1>  seconds

ratio<1, 1000>  microseconds

ratio<1, 1000000>  microseconds

ratio<1, 1000000000> nanosecons
```
这里需要说明一下ratio这个类模版的原型：
```cpp
template <intmax_t N, intmax_t D = 1> class ratio;
```
N代表分子，D代表分母，所以ratio表示一个分数值。

注意，我们自己可以定义Period，比如ratio<1, -2>表示单位时间是-0.5秒。

由于各种duration表示不同，chrono库提供了duration_cast类型转换函数。
```cpp
1 template <class ToDuration, class Rep, class Period>
2 constexpr ToDuration duration_cast (const duration<Rep,Period>& dtn);
```
典型的用法是表示一段时间：

```cpp

 1 // duration constructor
 2 #include <iostream>
 3 #include <ratio>
 4 #include <chrono>
 5  
 6 int main () 7 {
 8   typedef std::chrono::duration<int> seconds_type;
 9   typedef std::chrono::duration<int,std::milli> milliseconds_type; 
 10   typedef std::chrono::duration<int,std::ratio<60*60>> hours_type; 
 11  
12   hours_type h_oneday (24);                  // 24h
13   seconds_type s_oneday (60*60*24);          // 86400s
14   milliseconds_type ms_oneday (s_oneday);    // 86400000ms
15  
16   seconds_type s_onehour (60*60);            // 3600s 
17 //hours_type h_onehour (s_onehour); // NOT VALID (type truncates), use:
18   hours_type h_onehour (std::chrono::duration_cast<hours_type>(s_onehour)); 
19   milliseconds_type ms_onehour (s_onehour);  // 3600000ms (ok, no type truncation)
20  
21   std::cout << ms_onehour.count() << "ms in 1h" << std::endl; 
22  
23   return 0; 
24 } 
```
duration还有一个成员函数count()返回Rep类型的Period数量，看代码： 
```cpp
28 // duration::count
29 #include <iostream>     // std::cout
30 #include <chrono>       // std::chrono::seconds, std::chrono::milliseconds 
31                         // std::chrono::duration_cast
32  
33 int main () 
34 { 
35   using namespace std::chrono; 
36   // std::chrono::milliseconds is an instatiation of std::chrono::duration:
37   milliseconds foo (1000); // 1 second
38   foo*=60; 
39  
40   std::cout << "duration (in periods): "; 
41   std::cout << foo.count() << " milliseconds.\n"; 
42  
43   std::cout << "duration (in seconds): "; 
44   std::cout << foo.count() * milliseconds::period::num / milliseconds::period::den; 
45   std::cout << " seconds.\n"; 
46  
47   return 0; 
48 }
```
**_2.Time points_**

std::chrono::time_point 表示一个具体时间，如上个世纪80年代、你的生日、今天下午、火车出发时间等，只要它能用计算机时钟表示。鉴于我们使用时间的情景不同，这个time point具体到什么程度，由选用的单位决定。**一个time point必须有一个clock计时**。参见clock的说明。
```cpp
template <class Clock, class Duration = typename Clock::duration>  class time_point;
```
下面是构造使用time_point的例子：
```cpp
 1 // time_point constructors
 2 #include <iostream>
 3 #include <chrono>
 4 #include <ctime>
 5  
 6 int main () 7 {
 8   using namespace std::chrono; 
 9  
10   system_clock::time_point tp_epoch;    // epoch value
11  
12   time_point <system_clock,duration<int>> tp_seconds (duration<int>(1)); 
13  
14   system_clock::time_point tp (tp_seconds); 
15  
16   std::cout << "1 second since system_clock epoch = "; 
17   std::cout << tp.time_since_epoch().count(); 
18   std::cout << " system_clock periods." << std::endl; 
19  
20   // display time_point:
21   std::time_t tt = system_clock::to_time_t(tp); 
22   std::cout << "time_point tp is: " << ctime(&tt); 
23  
24   return 0; 
25 } 
26  
```

time_point有一个函数**time_from_eproch()**用来获得1970年1月1日到time_point时间经过的duration。

举个例子，如果timepoint以天为单位，函数返回的duration就以天为单位。

由于各种time_point表示方式不同，chrono也提供了相应的转换函数 time_point_cast。
```cpp
1 template <class ToDuration, class Clock, class Duration>
2 time_point<Clock,ToDuration> time_point_cast (const time_point<Clock,Duration>& tp);
```
比如计算

```cpp
 1 / time_point_cast 
 2 #include <iostream>
 3 #include <ratio>
 4 #include <chrono>
 5  
 6 int main () 
 7 {
 8   using namespace std::chrono; 
 9  
10   typedef duration<int,std::ratio<60*60*24>> days_type; 
11  
12   time_point<system_clock,days_type> today = time_point_cast<days_type>(system_clock::now()); 
13  
14   std::cout << today.time_since_epoch().count() << " days since epoch" << std::endl; 
15  
16   return 0; 
17 }
```

**3.Clocks**
chrono库定义了三种不同的时钟:
```cpp
1 std::chrono::system_clock:  依据系统的当前时间 (不稳定) 
2 std::chrono::steady_clock:  以统一的速率运行(不能被调整) 
3  std::chrono::high_resolution_clock: 提供最高精度的计时周期(可能是steady_clock或者system_clock的typedef)
```
这三个时钟类都提供了一个静态成员函数**now()**用于获取当前时间，该函数的返回值是一个time_point类型。

**std::chrono::system_clock** 它表示当前的系统时钟，系统中运行的所有进程使用now()得到的时间是一致的。就类似Windows系统右下角那个时钟，是系统时间。明显那个时钟是可以乱设置的。明明是早上10点，却可以设置成下午3点。

每一个clock类中都有确定的time_point, duration, Rep, Period类型。

操作有：
```cpp
now() 当前时间time_point

to_time_t() time_point转换成time_t秒
system_clock除了now()函数外，还提供了to_time_t()静态成员函数。
用于将系统时间转换成熟悉的std::time_t类型，得到了time_t类型的值，
在使用ctime()函数将时间转换成字符串格式，就可以很方便地打印当前时间了。

from_time_t() 从time_t转换成time_point
```

```cpp
#include<iostream>
#include<vector>
#include<string>
#include<ctime>//将时间格式的数据转换成字符串
#include<chrono>
using namespace std::chrono;
using namespace std;
int main()
{
    //获取系统的当前时间
    auto t = system_clock::now();
    //将获取的时间转换成time_t类型
    auto tNow = system_clock::to_time_t(t);

    //ctime()函数将time_t类型的时间转化成字符串格式,这个字符串自带换行符
    string str_time = std::ctime(&tNow);

    cout<<str_time;

    return 0;
}
```
典型的应用是计算时间日期：
```cpp
 1 // system_clock example
 2 #include <iostream>
 3 #include <ctime>
 4 #include <ratio>
 5 #include <chrono>
 6  
 7 int main () 
 8 {
 9   using std::chrono::system_clock; 
 10  
11   std::chrono::duration<int,std::ratio<60*60*24> > one_day (1); 12  
13   system_clock::time_point today = system_clock::now(); 
14   system_clock::time_point tomorrow = today + one_day; 
15  
16 std::time_t tt; 
17  
18   tt = system_clock::to_time_t ( today ); 
19   std::cout << "today is: " << ctime(&tt); 
20  
21   tt = system_clock::to_time_t ( tomorrow ); 
22   std::cout << "tomorrow will be: " << ctime(&tt); 
23  
24   return 0; 
25 } 
26  
```

**std::chrono::steady_clock** 为了表示稳定的时间间隔，后一次调用now()得到的时间总是比前一次的值大（这句话的意思其实是，如果中途修改了系统时间，也不影响now()的结果），每次tick都保证过了稳定的时间间隔。steady_clock则针对system_clock可以随意设置这个缺陷而提出来的，他表示时钟是不能设置的。

操作有：

now() 获取当前时钟

典型的应用是给算法计时：
```cpp

 1 // steady_clock example
 2 #include <iostream>
 3 #include <ctime>
 4 #include <ratio>
 5 #include <chrono>
 6  
 7 int main () 
 8 {
 9   using namespace std::chrono; 
 10  
11   steady_clock::time_point t1 = steady_clock::now(); 
12  
13   std::cout << "printing out 1000 stars...\n"; 
14   for (int i=0; i<1000; ++i) std::cout << "*"; 
15   std::cout << std::endl; 
16  
17   steady_clock::time_point t2 = steady_clock::now(); 
18  
19   duration<double> time_span = duration_cast<duration<double>>(t2 - t1); 
20  
21   std::cout << "It took me " << time_span.count() << " seconds."; 
22   std::cout << std::endl; 
23  
24   return 0; 
25 } 
26  
```

最后一个时钟，**std::chrono::high_resolution_clock** 顾名思义，这是系统可用的最高精度的时钟。实际上high_resolution_clock只不过是system_clock或者steady_clock的typedef。

操作有：
```cpp
now() 获取当前时钟。
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA5NzAwNDkwMiwtNzk2NzA5NTI5LDEwMz
kzOTg1NTQsMTQ2OTMwMDcyNSwtNDEwODI4MTQsLTExNjc1OTg0
OTQsMTM3OTI1NDQzMCwxMjE0NjU1MzQ4XX0=
-->