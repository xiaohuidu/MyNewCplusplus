`C++98`标准中并没有线程库的存在，直到`C++11`中才终于提供了多线程的标准库，提供了管理线程、保护共享数据、线程间同步操作、原子操作等类。多线程库对应的头文件是`#include <thread>`，类名为`std::thread`。

然而线程毕竟是比较贴近系统的东西，使用起来仍然不是很方便，特别是线程同步及获取线程运行结果上就更加麻烦。我们不能简单的通过`thread.join()`得到结果，必须定义一个线程共享的变量来传递结果，同时还要考虑线程间的互斥问题。好在`**C++11`中提供了一个相对简单的异步接口`std::async`，通过这个接口可以简单的创建线程并通过`std::future`中获取结果**以往都是自己去封装线程实现自己的async，现在有线程的跨平台接口可以使用就极大的方便了C++多线程编程。

先看一下`std::async`的函数原型
```
//(C++11 起) (C++17 前)
template< class Function, class... Args>
std::future<std::result_of_t<std::decay_t<Function>(std::decay_t<Args>...)>>
 async( Function&& f, Args&&... args );
 
//(C++11 起) (C++17 前)
template< class Function, class... Args >
std::future<std::result_of_t<std::decay_t<Function>(std::decay_t<Args>...)>>
 async( std::launch policy, Function&& f, Args&&... args );
 ```

**第一个参数是线程的创建策略，有两种策略可供选择**：

-   `std::launch::async`：在调用async就开始创建线程。
-   `std::launch::deferred`：延迟加载方式创建线程。调用async时不创建线程，直到调用了future的get或者wait时才创建线程。

默认策略是：`std::launch::async | std::launch::deferred`也就是两种策略的合集，具体什么意思后面详细再说

**第二个参数是线程函数**

线程函数可接受`function, lambda expression, bind expression, or another function object`

**第三个参数是线程函数的参数**

不再说明

**返回值std::future**

`std::future`是一个模板类，它提供了一种访问异步操作结果的机制。从字面意思上看它表示未来，这个意思就非常贴切，因为她不是立即获取结果但是可以在某个时候以同步的方式来获取结果。我们可以通过查询future的状态来获取异步操作的结构。future_status有三种状态：

-   deferred：异步操作还未开始
-   ready：异步操作已经完成
-   timeout：异步操作超时，主要用于std::future.wait_for()

**示例：**
```
`//查询future的状态
std::future_status status;
do {
 status = future.wait_for(std::chrono::seconds(1));
 if (status == std::future_status::deferred) {
 std::cout << "deferred" << std::endl;
 } else if (status == std::future_status::timeout) {
 std::cout << "timeout" << std::endl;
 } else if (status == std::future_status::ready) {
 std::cout << "ready!" << std::endl;
 }
} while (status != std::future_status::ready);` 

`std::future`获取结果的方式有三种：

-   get：等待异步操作结束并返回结果
-   wait：等待异步操作结束，但没有返回值
-   waite_for：超时等待返回结果，上面示例中就是对超时等待的使用展示

**介绍完了`std::async`的函数原型，那么它到底该如何使用呢？**

`std::async`的基本用法：[示例链接](https://link.juejin.cn?target=https%3A%2F%2Fwww.apiref.com%2Fcpp-zh%2Fcpp%2Fthread%2Fasync.html "https://www.apiref.com/cpp-zh/cpp/thread/async.html")

c

复制代码

`#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>
#include <future>
#include <string>
#include <mutex>
std::mutex m;
struct X {
 void foo(int i, const std::string& str) {
 std::lock_guard<std::mutex> lk(m);
 std::cout << str << ' ' << i << '\n';
 }
 void bar(const std::string& str) {
 std::lock_guard<std::mutex> lk(m);
 std::cout << str << '\n';
 }
 int operator()(int i) {
 std::lock_guard<std::mutex> lk(m);
 std::cout << i << '\n';
 return i + 10;
 }};
template <typename RandomIt>int parallel_sum(RandomIt beg, RandomIt end){
 auto len = end - beg;
 if (len < 1000)
 return std::accumulate(beg, end, 0);
 RandomIt mid = beg + len/2;
 auto handle = std::async(std::launch::async,
 parallel_sum<RandomIt>, mid, end);
 int sum = parallel_sum(beg, mid);
 return sum + handle.get();
}
int main(){
 std::vector<int> v(10000, 1);
 std::cout << "The sum is " << parallel_sum(v.begin(), v.end()) << '\n';
 X x;
 // 以默认策略调用 x.foo(42, "Hello") ：
 // 可能同时打印 "Hello 42" 或延迟执行
 auto a1 = std::async(&X::foo, &x, 42, "Hello");
 // 以 deferred 策略调用 x.bar("world!")
 // 调用 a2.get() 或 a2.wait() 时打印 "world!"
 auto a2 = std::async(std::launch::deferred, &X::bar, x, "world!");
 // 以 async 策略调用 X()(43) ：
 // 同时打印 "43"
 auto a3 = std::async(std::launch::async, X(), 43);
 a2.wait();                     // 打印 "world!"
 std::cout << a3.get() << '\n'; // 打印 "53"
} // 若 a1 在此点未完成，则 a1 的析构函数在此打印 "Hello 42"` 

可能的结果

python

复制代码

`The sum is 10000
43
world!
53
Hello 42` 

由此可见，`std::async`是异步操作做了一个很好的封装，使我们不用关注线程创建内部细节，就能方便的获取异步执行状态和结果，还可以指定线程创建策略。

**深入理解线程创建策略**

-   std::launch::async调度策略意味着函数必须异步执行，即在另一线程执行。
-   std::launch::deferred调度策略意味着函数可能只会在std::async返回的future对象调用get或wait时执行。那就是，执行会推迟到其中一个调用发生。当调用get或wait时，函数会同步执行，即调用者会阻塞直到函数运行结束。如果get或wait没有被调用，函数就绝对不会执行。

两者策略都很明确，然而该函数的默认策略却很有趣，它不是你显示指定的，也就是第一个函数原型中所用的策略即`std::launch::async | std::launch::deferred`，c++标准中给出的说明是：

> 进行异步执行还是惰性求值取决于实现

c

复制代码

`auto future = std::async(func);        // 使用默认发射模式执行func` 

这种调度策略我们没有办法预知函数func是否会在哪个线程执行，甚至无法预知会不会被执行，因为func可能会被调度为推迟执行，即调用get或wait的时候执行，而get或wait是否会被执行或者在哪个线程执行都无法预知。

同时这种调度策略的灵活性还会混淆使用thread_local变量，这意味着如果func写或读这种线程本地存储(Thread Local Storage，TLS)，预知取到哪个线程的本地变量是不可能的。

它也影响了基于wait循环中的超时情况，因为调度策略可能为`deferred`的，调用wait_for或者wait_until会返回值std::launch::deferred。这意味着下面的循环，看起来最终会停止，但是，实际上可能会一直运行：

c

复制代码

`void func()           // f睡眠1秒后返回
{
 std::this_thread::sleep_for(1);
}
auto future = std::async(func);      // （概念上）异步执行f
while(fut.wait_for(100ms) !=         // 循环直到f执行结束
 std::future_status::ready)     // 但这可能永远不会发生
{
 ...
}` 

为避免陷入死循环，我们必须检查future是否把任务推迟，然而future无法获知任务是否被推迟，一个好的技巧就是通过wait_for(0)来获取future_status是否是deferred：

c

复制代码

`auto future = std::async(func);      // （概念上）异步执行f
if (fut.wait_for(0) == std::future_status::deferred)  // 如果任务被推迟
{
 ...     // fut使用get或wait来同步调用f
} else {            // 任务没有被推迟
 while(fut.wait_for(100ms) != std::future_status::ready) { // 不可能无限循环
 ...    // 任务没有被推迟也没有就绪，所以做一些并发的事情直到任务就绪
 }
 ...        // fut就绪
}` 

有人可能会说既然有这么多缺点为啥还要用它，因为毕竟我们考虑的极限情况下的可能，有时候我不要求它是并发还是同步执行，也不需要考虑修改那个线程thread_local变量，同时也能接受可能任务永远不会执行，那么这种方式就是一种方便且高效的调度策略。

**综上所述，我们总结出以下几点：**

-   std::async的默认调度策略既允许任务异步执行，又允许任务同步执行。
-   默认策略灵活性导致了使用thread_local变量时的不确定性，它隐含着任务可能不会执行，它还影响了基于超时的wait调用的程序逻辑。
-   **如果异步执行是必需的，指定std::launch::async发射策略。**

  

作者：VE视频引擎  
链接：https://juejin.cn/post/6921601619758940174  
来源：稀土掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM4NjM0Njk0Niw3MzA5OTgxMTZdfQ==
-->