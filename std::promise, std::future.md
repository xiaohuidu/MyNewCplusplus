C++11中promise和future机制是用于并发编程的一种解决方案，**用于在不同线程完成数据传递（异步操作）**。

传统方式通过回调函数处理异步返回的结果，导致代码逻辑分散且难以维护。

**Promise和Future是一种提供访问异步操作结果的机制，可以在线程之间传递数据和异常消息**。

Promise和Future 在**不需要显式lock的情况下实现在两个不同的thread之间交换数据**。

**当一个thread 需要传递一个值给另一个thread， 它把值放到promise里， 底层实现让这个值出现在和这个promise 绑定的future里， 调用者可以读取这个值。**

应用场景：顾客在一家奶茶店点了单，服务员给顾客一个单号，当奶茶做好后，服务员更新排号的状态，顾客可以去做自己的事情了，顾客可以通过查询排号来得知奶茶是否做好，当查到奶茶做好了就可以回来取奶茶了。

```cpp
#include <iostream>
#include <future>
#include <thread>

using namespace std;
using namespace std::chrono;

void WaitForMilkTea(future<int>& future)
{
	/*其中获取future结果有三种方式
	 1、auto value = future.get()    get()方法会阻塞等待异步操作结束并返回结果
	 2、std::future_status  方式判断状态 有deferred、timeout、ready三种状态
	 3、可以
	*/

//future_status方法
#if 0
	std::future_status status;
	do {
		status = future.wait_for(std::chrono::milliseconds(500));
		if (status == std::future_status::deferred) {
			std::cout << "deferred!!!" << std::endl; //异步操作还没开始
		} else if (status == std::future_status::timeout) {
			std::cout << "timeout!!!" << std::endl; //异步操作超时
		} else if (status == std::future_status::ready) {
			std::cout << "ready!!!" << std::endl; //异步操作已经完成
		}
	} while (status != std::future_status::ready);
     
	//通过判断future_status状态为ready时才通过get()获取值
	auto notice = future.get();
	std::cout << "WaitForMilkTea receive value:" << notice << std::endl;
#endif

//get()方法
#if 0
	auto notice = future.get();   //get阻塞等待直到异步处理结束返回值
	std::cout << "WaitForMilkTea receive value:" << notice << std::endl;
#endif

//wait()方法
	future.wait(); //和get()区别是wait只等待异步操作完成，没有返回值
	auto notice = future.get();
	std::cout << "WaitForMilkTea receive value:" << notice << std::endl;
}

void MakeTea(promise<int>& promise)
{
	//do something 这里先睡眠5s
	std::this_thread::sleep_for(std::chrono::seconds(5));
	promise.set_value(1);
	std::this_thread::sleep_for(std::chrono::seconds(10));
	std::cout << "MakeTea Thread quit!!!" << std::endl;
}

int main()
{
	promise<int> pNotice;
	//获取与promise相关联的future
	future<int> pFuture = pNotice.get_future();

	thread Customer(WaitForMilkTea, ref(pFuture));
	thread Worker(MakeTea, ref(pNotice));

	Customer.join();
	Worker.join();
}
```

其中future_status枚举如下：
名称	值	示意
ready	0	就绪
timeout	1	等待超时
deferred	2	延迟执行(与std::async配合使用)


future_errc 枚举 : 为 future_error 类报告的所有错误提供符号名称。
名称	值	示意
broken_promise	0	与其关联的 std::promise 生命周期提前结束。
future_already_retrieved	1	重复调用 get() 函数。
promise_already_satisfied	2	与其关联的 std::promise 重复 set。
no_state	4	无共享状态。

**Promise和Future模型 流程如下：**

- 线程1初始化一个promise， 并从promose 获得future对象，将promise对象传递给线程2，相当于线程2对线程1的一个承诺。
- future用于获取未来线程2的值
- 线程2接受一个promise，需要将处理结果通过promise返回给线程1
- 线程1想要获取数据，此时线程2还未返回promise就阻塞等待处，直到线程2的数据可达

**promise相关函数**
std::future负责访问， std::future是一个模板类，它提供了可供访问异步执行结果的一种方式。

std::promise负责存储， std::promise也是一个模板类，它提供了存储异步执行结果的值和异常的一种方式。

**总结：std::future负责访问，std::promise负责存储，同时promise是future的管理者**

### std::future

|名称  |作用  |
|--|--|
|operator=  |移动 future 对象，**移动**！  |
|share()  |回一个可在多个线程中共享的 std::shared_future 对象  |
|valid()  |检查 future 是否处于被使用状态，也就是它被首次在首次调用 get() 或 share() 前。建议使用前加上valid()判断  |
|wait()  |阻塞等待调用它的线程到共享值成功返回。  |
|wait_for()  |在规定时间内 阻塞等待调用它的线程到共享值成功返回。  |
|wait_until()	  |在指定时间节点内 阻塞等待调用它的线程到共享值成功返回。  |

- std::future仅在创建它的std::promise(或者std::async、std::packaged_task)有效时才有用，所以可以在使用前用valid()判断

- std::future可供异步操作创建者用各种方式查询、等待、提取需要共享的值，也可以阻塞当前线程等待到异步线程提供值。
-  std::future一个实例只能与一个异步线程相关联，多个线程则需要使用std::shared_future。


### std::promise
|名称  |作用  |
|--|--|
|operator=  |从另一个 std::promise 移动到当前对象。  |
|swap()	  |交换移动两个 std::promise。  |
|get_future()  |获取与其管理的std::future  |
|set_value()	  |设置共享状态值，此后promise共享状态标识变为ready。  |
|set_value_at_thread_exit()  |设置共享状态的值，但是不将共享状态的标志设置为 ready，当线程退出时该 promise 对象会自动设置为 ready  |
|set_exception()  |设置异常，此后promise的共享状态标识变为ready。  |
|set_exception_at_thread_exit()  |设置异常，但是到该线程结束时才会发出通知。  |

- std::promise::get_future：返回一个与promise共享状态相关联的future对象
- std::promise::set_value：设置共享状态的值，此后promise共享状态标识变为ready
- std::promise::set_exception：为promise设置异常，此后promise的共享状态标识变为ready
- std::promise::set_value_at_thread_exit：设置共享状态的值，但是不将共享状态的标志设置为 ready，当线程退出时该 promise 对象会自动设置为 ready（注意：该线程已设置promise的值，如果在线程结束之后有其他修改共享状态值的操作，会抛出future_error(promise_already_satisfied)异常）

- std::promise::swap：交换 promise 的共享状态

std::promise的**set相关函数和get_future()只能被调用一次**，多次调用会抛出异常

std::promise作为使用者的异步线程中应当注意到共享变量的生命周期、是否被set问题。如果没被set而线程就结束了，future端就会抛出异常。

### 多线程std::shared_future
std::future 有个非常明显的问题，就是只能和一个 std::promise 成对绑定使用，也就意味着仅限于两个线程之间使用。

那么**多个线程**是否可以呢，可以！就是 std::shared_future。

std::shared_future 也是一个模板类，它的功能定位、函数接口和 std::future 一致，不同的是它允许给多个线程去使用，让多个线程去同步、共享：

它的语法是：

**【语法】std::shared_future<Type> s_fu(pt.get_future());**
```
#include <iostream>
#include <future>
#include <thread>

using namespace std;
using namespace std::chrono;

void futureHandleEntry(std::shared_future<int>& future) 
{
	if (future.valid()) {
		future.wait();
		std::cout << "thread:[" << std::this_thread::get_id() << "] value=" << future.get() << std::endl;
		std::cout << "thread:[" << std::this_thread::get_id() << "] quit!!!" << std::endl;
	}
	else {
		std::cout << "future is invalid!!!" << std::endl;
	}
}

int main()
{
    std::promise<int> promise;
	//获取shared_future用于多线程访问异步操作结果
	std::shared_future<int> future = promise.get_future();

	std::thread t1(&futureHandleEntry, ref(future));
	std::thread t2(&futureHandleEntry, ref(future));
	std::thread t3(&futureHandleEntry, ref(future));

	std::cout << "main thread running!!!" << std::endl;
	//主线程sleep5s后去set_value
	std::this_thread::sleep_for(std::chrono::seconds(5));
	promise.set_value(10);

	t1.join();
	t2.join();
	t3.join();
}

```

### promise和future进阶
我们知道异常的场景：

**1、当重复调用promise的set_value会导致抛出异常**
```
#include <iostream>
#include <thread>
#include <future>
#include <chrono>

using namespace std;

void threadEntry(std::future<int>& future)
{
	try {
		auto value = future.get();
		std::cout << "value=" << value << std::endl;
	}
	catch (std::future_error& error) {
		std::cerr << error.code() << "\n" << error.what() << std::endl;
	}
}
int main()
{
	std::promise<int> promise;
	std::future<int> future = promise.get_future();

	std::thread t1(threadEntry, ref(future));
	/*主线程promise多次调用set_value会抛出future_error异常
	*/
	std::this_thread::sleep_for(std::chrono::seconds(2));
	promise.set_value(1); 
	promise.set_value(1);

	t1.join();
}
```

在linux中运行结果如下： 会有Promise already satisfied的错误提示

**2、 当std::promise不设置值时线程就退出**
如果promise直到销毁时，都未设置过任何值，则promise会在析构时自动设置为std::future_error，这**会造成std::future.get抛出std::future_error异常**。
```
#include <iostream> // std::cout, std::endl
 2#include <thread>   // std::thread
 3#include <future>   // std::promise, std::future
 4#include <chrono>   // seconds
 5using namespace std::chrono;
 6
 7void read(std::future<int> future) {
 8    try {
 9        future.get();
10    } catch(std::future_error &e) {
11        std::cerr << e.code() << "\n" << e.what() << std::endl;
12    }
13}
14
15int main() {
16    std::thread thread;
17    {
18        // 如果promise不设置任何值
19        // 则在promise析构时会自动设置为future_error
20        // 这会造成future.get抛出该异常
21        std::promise<int> promise;
22        thread = std::thread(read, promise.get_future());
23    }
24    thread.join();
25
26    return 0;
27}
```

**3、通过std::promise让std::future抛出异常**

通过std::promise.set_exception函数可以设置自定义异常，该异常最终会被传递到std::future，并在其get函数中被抛出。
```
#include <iostream>
 #include <future>
 #include <thread>
 #include <exception>  // std::make_exception_ptr
 #include <stdexcept>  // std::logic_error
 
void catch_error(std::future<void> &future) {
    try {
       future.get();
    } catch (std::logic_error &e) {
       std::cerr << "logic_error: " << e.what() << std::endl;
   }
}

int main() {
    std::promise<void> promise;
    std::future<void> future = promise.get_future();

    std::thread thread(catch_error, std::ref(future));
    // 自定义异常需要使用make_exception_ptr转换一下
    promise.set_exception(
       std::make_exception_ptr(std::logic_error("caught")));

      thread.join();
     return 0;
}
```

**std::promise虽然支持自定义异常，但它并不直接接受异常对象：**

// std::promise::set_exception函数原型
void **set_exception**(std::exception_ptr p);

自定义异常可以通过位于头文件exception下的**std::make_exception_ptr**函数转化为std::exception_ptr


**std::promise<void> 是合法的**。此时std::promise.set_value不接受任何参数，仅用于通知关联的std::future.get()解除阻塞。

std::async(异步运行)时，开发人员有时会对std::promise所在线程退出时间比较关注。**std::promise支持定制线程退出时的行为**：

std::**promise.set_value_at_thread_exit** 线程退出时，std::future收到通过该函数设置的值
std::**promise.set_exception_at_thread_exit** 线程退出时，std::future则抛出该函数指定的异常。

**packaged_task** 可以用来简单的启动一个线程， 它会负责创建 future 和 promise。

```
 double comp(vector<double>& v)
	{
		//** package the tasks**:
		//** (the task here is the standard accumulate() for an array of doubles)**:
		packaged_task<double(double*,double*,double)> pt0{std::accumulate<double*,double*,double>};
		packaged_task<double(double*,double*,double)> pt1{std::accumulate<double*,double*,double>};

		auto f0 = pt0.get_future();	//** get hold of the futures **auto f1 = pt1.get_future();

		pt0(&v[0],&v[v.size()/2],0);	//** start the threads **pt1(&v[v.size()/2],&v[size()],0);
	
		return f0.get()+f1.get();	//** get the results **}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY5MTE5NjQ4MiwtMzMxMzc1MDQyLDE4MT
c3OTkwNDIsNTI2MTkwMjQ1LC0xOTExNjMyMjAxLDc4MzU3MTIs
LTI2OTE2NjgwNSwtNDgwMzQwNzIxLC0xMzU5NzAwMzMyLC00OD
AzNDA3MjEsMTE5OTcxMTM1M119
-->