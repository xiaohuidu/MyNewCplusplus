## 1、多线程

传统的C++（C++11标准之前）中并没有引入线程这个概念，在C++11出来之前，如果我们想要在C++中实现多线程，需要借助操作系统平台提供的API，比如Linux的<pthread.h>，或者windows下的<windows.h> 。

C++11提供了语言层面上的多线程，包含在头文件<thread>中。它解决了跨平台的问题，提供了管理线程、保护共享数据、线程间同步操作、原子操作等类。C++11 新标准中引入了5个头文件来支持多线程编程，如下图所示：

![](https://pic2.zhimg.com/80/v2-76e5e48c9c1d60f9868452cfc9ce7d85_720w.webp)


### 1.1、创建线程

创建线程很简单，只需要把函数添加到线程当中即可。

-   形式1：

```cpp
std::thread myThread ( thread_fun);
//函数形式为void thread_fun()
myThread.join();
//同一个函数可以代码复用，创建多个线程
```

使用lambda表达式启动线程输出数字
    
 ```cpp
    for (int i = 0; i < 4; i++)
    {
    	thread t([i]{
    		cout << i << endl;
    	});
    	t.detach();
    }
```
-   形式2：

```cpp
std::thread myThread ( thread_fun(100));
myThread.join();
//函数形式为void thread_fun(int x)
//同一个函数可以代码复用，创建多个线程
```
使用重载了()运算符的类实现多线程数字输出


```cpp
class Task
{
public:
	void operator()(int i)
	{
		cout << i << endl;
	}
};

int main()
{
	
	for (uint8_t i = 0; i < 4; i++)
	{
		Task task;
		thread t(task, i);
		t.detach();	
	}
}
```

-   形式3：

```cpp
std::thread (thread_fun,1).detach();
//直接创建线程，没有名字
//函数形式为void thread_fun(int x)
```

std::thread (thread_fun,1).detach();

-   代码实例:

使用g++编译下列代码的方式：

> g++  [http://test.cc](https://link.zhihu.com/?target=http%3A//test.cc)  -o test -l pthread

```text
#include <iostream>
#include <thread>
using namespace std;

void thread_1()
{
    cout<<"子线程1"<<endl;
}

void thread_2(int x)
{
    cout<<"x:"<<x<<endl;
    cout<<"子线程2"<<endl;
}

int main()
{
  thread first ( thread_1); // 开启线程，调用：thread_1()
  thread second (thread_2,100); // 开启线程，调用：thread_2(100)
  //thread third(thread_2,3);//开启第3个线程，共享thread_2函数。
  std::cout << "主线程\n";

  first.join(); //必须说明添加线程的方式            
  second.join(); 
  std::cout << "子线程结束.\n";//必须join完成
  return 0;
}
```

### 1.2、join与detach方式

当线程启动后，一定要在和线程相关联的thread销毁前，确定以何种方式等待线程执行结束。比如上例中的join。

-   detach方式，启动的线程自主在后台运行，当前的代码继续往下执行，不等待新线程结束。
-   join方式，等待启动的线程完成，才会继续往下执行。

可以使用joinable判断是join模式还是detach模式。

```text
if (myThread.joinable()) foo.join();
```

（1）join举例

下面的代码，join后面的代码不会被执行，除非子线程结束。

```text
#include <iostream>
#include <thread>
using namespace std;
void thread_1()
{
  while(1)
  {
  //cout<<"子线程1111"<<endl;
  }
}
void thread_2(int x)
{
  while(1)
  {
  //cout<<"子线程2222"<<endl;
  }
}
int main()
{
    thread first ( thread_1); // 开启线程，调用：thread_1()
    thread second (thread_2,100); // 开启线程，调用：thread_2(100)

    first.join(); // pauses until first finishes 这个操作完了之后才能destroyed
    second.join(); // pauses until second finishes//join完了之后，才能往下执行。
    while(1)
    {
      std::cout << "主线程\n";
    }
    return 0;
}
```

### （2）detach举例

下列代码中，主线程不会等待子线程结束。如果主线程运行结束，程序则结束。

```text
#include <iostream>
#include <thread>
using namespace std;

void thread_1()
{
  while(1)
  {
      cout<<"子线程1111"<<endl;
  }
}

void thread_2(int x)
{
    while(1)
    {
        cout<<"子线程2222"<<endl;
    }
}

int main()
{
    thread first ( thread_1);  // 开启线程，调用：thread_1()
    thread second (thread_2,100); // 开启线程，调用：thread_2(100)

    first.detach();                
    second.detach();            
    for(int i = 0; i < 10; i++)
    {
        std::cout << "主线程\n";
    }
    return 0;
}
```
### 异常情况下等待线程完成

当决定以detach方式让线程在后台运行时，可以在创建`thread`的实例后立即调用`detach`，这样线程就会后`thread`的实例分离，即使出现了异常`thread`的实例被销毁，仍然能保证线程在后台运行。

但线程以join方式运行时，需要在主线程的合适位置调用`join`方法，如果调用`join`前出现了异常，`thread`被销毁，线程就会被异常所终结。为了避免异常将线程终结，或者由于某些原因，例如线程访问了局部变量，就要保证线程一定要在函数退出前完成，就要保证要在函数退出前调用`join`

```cpp
void func() {
	thread t([]{
		cout << "hello C++ 11" << endl;
	});

	try
	{
		do_something_else();
	}
	catch (...)
	{
		t.join();
		throw;
	}
	t.join();
}
```

上面代码能够保证在正常或者异常的情况下，都会调用`join`方法，这样线程一定会在函数`func`退出前完成。但是使用这种方法，不但代码冗长，而且会出现一些作用域的问题，并不是一个很好的解决方法。

一种比较好的方法是**资源获取即初始化（RAII**,Resource Acquisition Is Initialization)，该方法提供一个类，在析构函数中调用`join`。


```cpp
class thread_guard
{
	thread &t;
public :
	explicit thread_guard(thread& _t) :
		t(_t){}

	~thread_guard()
	{
		if (t.joinable())
			t.join();
	}

	thread_guard(const thread_guard&) = delete;
	thread_guard& operator=(const thread_guard&) = delete;
};

void func(){

	thread t([]{
		cout << "Hello thread" <<endl ;
	});

	thread_guard g(t);
}
```

无论是何种情况，当函数退出时，局部变量`g`调用其析构函数销毁，从而能够保证`join`一定会被调用。

### 向线程传递参数

向线程调用的函数传递参数也是很简单的，只需要在构造`thread`的实例时，依次传入即可。例如：

cpp

```cpp
void func(int *a,int n){}

int buffer[10];
thread t(func,buffer,10);
t.join();
```

需要注意的是，**默认的会将传递的参数以拷贝的方式复制到线程空间，即使参数的类型是引用。**例如：

cpp

```cpp
void func(int a,const string& str);
thread t(func,3,"hello");
```

`func`的第二个参数是`string &`，而传入的是一个字符串字面量。该字面量以`const char*`类型传入线程空间后，在**线程的空间内转换为`string`**。

如果在线程中使用引用来更新对象时，就需要注意了。默认的是将对象拷贝到线程空间，其**引用的是拷贝的线程空间的对象，而不是初始希望改变的对象**。如下：

```cpp
class _tagNode
{
public:
	int a;
	int b;
};

void func(_tagNode &node)
{
	node.a = 10;
	node.b = 20;
}

void f()
{
	_tagNode node;

	thread t(func, node);
	t.join();

	cout << node.a << endl ;
	cout << node.b << endl ;
}
```

在线程内，将对象的字段a和b设置为新的值，但是在线程调用结束后，这两个字段的值并不会改变。这样由于引用的实际上是局部变量`node`的一个拷贝，而不是`node`本身。在将对象传入线程的时候，调用`std::ref`，将`node`的引用传入线程，而不是一个拷贝。例如：`thread t(func,std::ref(node));`

也可以使用类的成员函数作为线程函数，示例如下

cpp

```cpp
class _tagNode{

public:
	void do_some_work(int a);
};
_tagNode node;

thread t(&_tagNode::do_some_work, &node,20);
```

上面创建的线程会调用`node.do_some_work(20)`，第三个参数为成员函数的第一个参数，以此类推。

### 转移线程的所有权")转移线程的所有权

**`thread`是可移动的(movable)的，但不可复制(copyable)**。可以通过`move`来改变线程的所有权，灵活的决定线程在什么时候join或者detach。

cpp

```cpp
thread t1(f1);
thread t3(move(t1));
```

将线程从t1转移给t3,这时候t1就不再拥有线程的所有权，调用`t1.join`或`t1.detach`会出现异常，要使用t3来管理线程。这也就意味着`thread`可以作为函数的返回类型，或者作为参数传递给函数，能够更为方便的管理线程。

线程的标识类型为`std::thread::id`，有两种方式获得到线程的id。

-   通过`thread`的实例调用`get_id()`直接获取
-   在当前线程上调用`this_thread::get_id()`获取


### 1.3、this_thread

this_thread是一个类，它有4个功能函数，具体如下：

- std::this_thread::get_id()：获取线程id
- std::this_thread::yield()：放弃线程执行，回到就绪状态
- std::this_thread::sleep_for(std::chrono::seconds(1))： 暂停1秒
- this_thread::sleep_until ：暂停直到某一刻

```cpp
using std::chrono::system_clock;
std::time_t tt = system_clock::to_time_t(system_clock::now());
struct std::tm * ptm = std::localtime(&tt);
cout << "Waiting for the next minute to begin...\n";
++ptm->tm_min; //加一分钟
ptm->tm_sec = 0; //秒数设置为0//暂停执行，到下一整分执行
this_thread::sleep_until(system_clock::from_time_t(mktime(ptm)));
```

## 2、mutex

mutex头文件主要声明了与互斥量(mutex)相关的类。mutex提供了4种互斥类型，如下表所示。

- std::mutex： 最基本的 Mutex 类。
- std::recursive_mutex：递归 Mutex 类。
- std::time_mutex：定时 Mutex 类。
- std::recursive_timed_mutex：定时递归 Mutex 类。

std::mutex 是C++11 中最基本的互斥量，std::mutex 对象提供了独占所有权的特性——即不支持递归地对 std::mutex 对象上锁，而 std::recursive_lock 则可以递归地对互斥量对象上锁。

### 2.1、lock与unlock

mutex常用操作：

-   lock()：资源上锁
-   unlock()：解锁资源
-   trylock()：查看是否上锁，它有下列3种类情况：

（1）未上锁返回false，并锁住；

（2）其他线程已经上锁，返回true；

（3）同一个线程已经对它上锁，将会产生死锁。

下面结合实例对lock和unlock进行说明。

同一个mutex变量上锁之后，一个时间段内，只允许一个线程访问它。例如：

```cpp
#include <iostream>  // std::cout
#include <thread>  // std::thread
#include <mutex>  // std::mutex

std::mutex mtx;  // mutex for critical section
void print_block (int n, char c) 
{
// critical section (exclusive access to std::cout signaled by locking mtx):
    mtx.lock();
    for (int i=0; i<n; ++i) 
    {
       std::cout << c; 
    }
    std::cout << '\n';
    mtx.unlock();
}
int main ()
{
    std::thread th1 (print_block,50,'');//线程1：打印*
    std::thread th2 (print_block,50,'$');//线程2：打印$

    th1.join();
    th2.join();
    return 0;
}
```

输出：

```text
**************************************************
$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
```

如果是不同mutex变量，因为不涉及到同一资源的竞争，所以以下代码运行可能会出现交替打印的情况，或者另一个线程可以修改共同的全局变量！

```text
#include <iostream> // std::cout
#include <thread> // std::thread
#include <mutex> // std::mutex

std::mutex mtx_1; // mutex for critical section
std::mutex mtx_2;  // mutex for critical section
int test_num = 1;

void print_block_1 (int n, char c) 
{
// critical section (exclusive access to std::cout signaled by locking mtx):
    mtx_1.lock();
    for (int i=0; i<n; ++i) 
    {
        //std::cout << c;
        test_num = 1;
        std::cout<<test_num<<std::endl;
    }
        std::cout << '\n';
    mtx_1.unlock();
}

void print_block_2 (int n, char c) 
{// critical section (exclusive access to std::cout signaled by locking mtx):
    mtx_2.lock();
    test_num = 2;
    for (int i=0; i<n; ++i) 
    {
    //std::cout << c;
      test_num = 2;
      std::cout<<test_num<<std::endl;
    }
    mtx_2.unlock();
}

int main ()
{
    std::thread th1 (print_block_1,10000,'*');
    std::thread th2 (print_block_2,10000,'$');

    th1.join();
    th2.join();
    return 0;
}
```

### 2.2、lock_guard

创建lock_guard对象时，它将尝试获取提供给它的互斥锁的所有权。当控制流离开lock_guard对象的作用域时，lock_guard析构并释放互斥量。lock_guard的特点：

-   创建即加锁，作用域结束自动析构并解锁，无需手工解锁
-   不能中途解锁，必须等作用域结束才解锁
-   不能复制

代码举例

```text
#include <thread>
#include <mutex>
#include <iostream>
int g_i = 0;
std::mutex g_i_mutex;  // protects g_i，用来保护g_i

void safe_increment()
{
    const std::lock_guard<std::mutex> lock(g_i_mutex);
    ++g_i;
    std::cout << std::this_thread::get_id() << ": " << g_i << '\n';// g_i_mutex自动解锁}int main(){
    std::cout << "main id: " <<std::this_thread::get_id()<<std::endl;
    std::cout << "main: " << g_i << '\n';

    std::thread t1(safe_increment);
    std::thread t2(safe_increment);

    t1.join();
    t2.join();

    std::cout << "main: " << g_i << '\n';
}
```

说明：

1.  该程序的功能为，每经过一个线程，g_i 加1。
2.  因为涉及到共同资源g_i ，所以需要一个共同mutex：g_i_mutex。
3.  main线程的id为1，所以下次的线程id依次加1。

### 2.3、unique_lock

简单地讲，unique_lock 是 lock_guard 的升级加强版，它具有 lock_guard 的所有功能，同时又具有其他很多方法，使用起来更加灵活方便，能够应对更复杂的锁定需要。unique_lock的特点：

-   创建时可以不锁定（通过指定第二个参数为std::defer_lock），而在需要时再锁定
-   可以随时加锁解锁
-   作用域规则同 lock_grard，析构时自动释放锁
-   **不可复制，可移动**
-   条件变量需要该类型的锁作为参数（此时必须使用unique_lock）

所有 lock_guard 能够做到的事情，都可以使用 unique_lock 做到，反之则不然。那么何时使lock_guard呢？很简单，**需要使用锁的时候，首先考虑使用 lock_guard，因为lock_guard是最简单的锁**。

下面是代码举例：

```cpp
#include <mutex>
#include <thread>
#include <iostream>
struct Box {
    explicit Box(int num) : num_things{num} {}
    int num_things;
    std::mutex m;
};
void transfer(Box &from, Box &to, int num)
{
    // defer_lock表示暂时unlock，默认自动加锁
    std::unique_lock<std::mutex> lock1(from.m, std::defer_lock);
    std::unique_lock<std::mutex> lock2(to.m, std::defer_lock);//两个同时加锁
    std::lock(lock1, lock2);//或者使用lock1.lock()

    from.num_things -= num;
    to.num_things += num;//作用域结束自动解锁,也可以使用lock1.unlock()手动解锁
}
int main()
{
    Box acc1(100);
    Box acc2(50);

    std::thread t1(transfer, std::ref(acc1), std::ref(acc2), 10);
    std::thread t2(transfer, std::ref(acc2), std::ref(acc1), 5);

    t1.join();
    t2.join();
    std::cout << "acc1 num_things: " << acc1.num_things << std::endl;
    std::cout << "acc2 num_things: " << acc2.num_things << std::endl;
}
```

说明：

-   该函数的作用是，从一个结构体中的变量减去一个num，加载到另一个结构体的变量中去。
-   std::mutex m;在结构体中，mutex不是共享的。但是只需要一把锁也能锁住，因为引用传递后，同一把锁传给了两个函数。
-   cout需要在join后面进行，要不然cout的结果不一定是最终算出来的结果。
-   std::ref 用于包装按引用传递的值。
-   std::cref 用于包装按const引用传递的值。

## 3、condition_variable

condition_variable头文件有两个variable类，一个是condition_variable，另一个是condition_variable_any。**condition_variable必须结合unique_lock使用**。condition_variable_any可以使用任何的锁。

下面以condition_variable为例进行介绍。

condition_variable条件变量可以阻塞（wait、wait_for、wait_until）调用的线程直到使用（notify_one或notify_all）通知恢复为止。condition_variable是一个类，这个类既有构造函数也有析构函数，使用时需要构造对应的condition_variable对象，调用对象相应的函数来实现上面的功能。

- wait:  Wait until notified
- wait_for:  Wait for timeout or until notified
- wait_until:  Wait until notified or time point
- notify_one: 解锁一个线程，如果有多个，则未知哪个线程执行
- notify_all:  解锁所有线程
- cv_status: 这是一个类，表示variable 的状态，如下所示

```cpp
enum class cv_status { no_timeout, timeout };
```

### 3.1、wait

当前线程调用 wait() 后将被阻塞(此时当前线程应该获得了锁（mutex），不妨设获得锁 lck)，直到另外某个线程调用 notify_* 唤醒了当前线程。在线程被阻塞时，该函数会自动调用 lck.unlock() 释放锁，使得其他被阻塞在锁竞争上的线程得以继续执行。另外，一旦当前线程获得通知(notified，通常是另外某个线程调用 notify_* 唤醒了当前线程)，wait()函数也是自动调用 lck.lock()，使得lck的状态和 wait 函数被调用时相同。代码示例：

```cpp
#include <iostream>           // std::cout
#include <thread>             // std::thread, std::this_thread::yield
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable

std::mutex mtx;
std::condition_variable cv;
int cargo = 0;
bool shipment_available() 
{
    return cargo!=0;
}
void consume (int n) 
{
    for (int i=0; i<n; ++i) 
    {
        std::unique_lock<std::mutex> lck(mtx);//自动上锁
        //第二个参数为false才阻塞（wait），阻塞完即unlock，给其它线程资源
        cv.wait(lck,shipment_available);// consume:
        std::cout << cargo << '\n';
        cargo=0;
    }
}
int main ()
{
    std::thread consumer_thread (consume,10);
    for (int i=0; i<10; ++i) 
    {
        //每次cargo每次为0才运行。
        while (shipment_available())  std::this_thread::yield();
        std::unique_lock<std::mutex> lck(mtx);
        cargo = i+1;
        cv.notify_one();
    }

    consumer_thread.join();
    return 0;
}

```

说明：

1.  主线程中的while，每次在cargo=0才运行。
2.  每次cargo被置为0，会通知子线程unblock(非阻塞)，也就是子线程可以继续往下执行。
3.  子线程中cargo被置为0后，wait又一次启动等待。也就是说shipment_available为false，则等待。

### 3.2、wait_for

与std::condition_variable::wait() 类似，不过 wait_for可以指定一个时间段，在当前线程收到通知或者指定的时间 rel_time 超时之前，该线程都会处于阻塞状态。而一旦超时或者收到了其他线程的通知，wait_for返回，剩下的处理步骤和 wait()类似。

```cpp
template <class Rep, class Period>
  cv_status wait_for (unique_lock<mutex>& lck,
                      const chrono::duration<Rep,Period>& rel_time);
```

另外，wait_for 的重载版本的最后一个参数pred表示 wait_for的预测条件，只有当 pred条件为false时调用 wait()才会阻塞当前线程，并且在收到其他线程的通知后只有当 pred为 true时才会被解除阻塞。

```cpp
template <class Rep, class Period, class Predicate>
    bool wait_for (unique_lock<mutex>& lck,
         const chrono::duration<Rep,Period>& rel_time, Predicate pred);
```

代码示例：

```cpp
#include <iostream>           // std::cout
#include <thread>             // std::thread
#include <chrono>             // std::chrono::seconds
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable, std::cv_status

std::condition_variable cv;
int value;
void read_value() 
{
    std::cin >> value;
    cv.notify_one();
}
int main ()
{
    std::cout << "Please, enter an integer (I'll be printing dots): \n";
    std::thread th (read_value);
  
    std::mutex mtx;
    std::unique_lock<std::mutex> lck(mtx);
    while (cv.wait_for(lck,std::chrono::seconds(1))==std::cv_status::timeout) 
    {
        std::cout << '.' << std::endl;
    }
    std::cout << "You entered: " << value << '\n';

    th.join();
    return 0;
}
```

-   通知或者超时都会解锁，所以主线程会一直打印。
-   示例中只要过去一秒，就会不断的打印。

## 4、线程池

### 4.1、概念

在一个程序中，如果我们需要多次使用线程，这就意味着，需要多次的创建并销毁线程。而创建并销毁线程的过程势必会消耗内存，线程过多会带来调动的开销，进而影响缓存局部性和整体性能。线程的创建并销毁有以下一些缺点：

-   创建太多线程，将会浪费一定的资源，有些线程未被充分使用。
-   销毁太多线程，将导致之后浪费时间再次创建它们。
-   创建线程太慢，将会导致长时间的等待，性能变差。
-   销毁线程太慢，导致其它线程资源饥饿。

线程池维护着多个线程，这避免了在处理短时间任务时，创建与销毁线程的代价。

### 4.2、线程池的实现

因为程序边运行边创建线程是比较耗时的，所以我们通过池化的思想：在程序开始运行前创建多个线程，这样，程序在运行时，只需要从线程池中拿来用就可以了．大大提高了程序运行效率．一般线程池都会有以下几个部分构成：

1.  线程池管理器（ThreadPoolManager）:用于创建并管理线程池，也就是线程池类
2.  工作线程（WorkThread）: 线程池中线程
3.  任务队列task: 用于存放没有处理的任务。提供一种缓冲机制。
4.  append：用于添加任务的接口

线程池实现代码：

```cpp
#ifndef _THREADPOOL_H
#define _THREADPOOL_H
#include <vector>
#include <queue>
#include <thread>
#include <iostream>
#include <stdexcept>
#include <condition_variable>
#include <memory> //unique_ptr
#include<assert.h>
const int MAX_THREADS = 1000; //最大线程数目
template <typename T>
class threadPool
{
public:
    threadPool(int number = 1);//默认开一个线程
    ~threadPool();
    std::queue<T > tasks_queue; //任务队列
    bool append(T *request);//往请求队列＜task_queue＞中添加任务<T >
private:
//工作线程需要运行的函数,不断的从任务队列中取出并执行
    static void *worker(void arg);
    void run();
private:
    std::vector<std::thread> work_threads; //工作线程

    std::mutex queue_mutex;
    std::condition_variable condition;  //必须与unique_lock配合使用
    bool stop;
};//end class//构造函数，创建线程
template <typename T>
threadPool<T>::threadPool(int number) : stop(false)
{
    if (number <= 0 || number > MAX_THREADS)
        throw std::exception();
    for (int i = 0; i < number; i++)
    {
        std::cout << "created Thread num is : " << i <<std::endl;
        work_threads.emplace_back(worker, this);
        //添加线程
        //直接在容器尾部创建这个元素，省去了拷贝或移动元素的过程。
    }
}
template <typename T>
inline threadPool<T>::~threadPool()
{
    std::unique_lock<std::mutex> lock(queue_mutex);
    stop = true;
    
    condition.notify_all();
    for (auto &ww : work_threads)
        ww.join();//可以在析构函数中join
}
//添加任务
template <typename T>
bool threadPool<T>::append(T *request)
{
    //操作工作队列时一定要加锁，因为他被所有线程共享
    queue_mutex.lock();//同一个类的锁
    tasks_queue.push(request);
    queue_mutex.unlock();
    condition.notify_one();  //线程池添加进去了任务，自然要通知等待的线程
    return true;
}//单个线程
template <typename T>
void threadPool<T>::worker(void *arg)
{
    threadPool pool = (threadPool *)arg;
    pool->run();//线程运行
    return pool;
}
template <typename T>
void threadPool<T>::run()
{
while (!stop)
{
    std::unique_lock<std::mutex> lk(this->queue_mutex);
    /*　unique_lock() 出作用域会自动解锁　/
    this->condition.wait(lk, [this] 
    { 
      return !this->tasks_queue.empty(); 
    });//如果任务为空，则wait，就停下来等待唤醒//需要有任务，才启动该线程，不然就休眠
    if (this->tasks_queue.empty())//任务为空，双重保障
    {  
        assert(0&&"断了");//实际上不会运行到这一步，因为任务为空，wait就休眠了。
        continue;
    }else{
        T *request = tasks_queue.front();
        tasks_queue.pop();
        if (request)//来任务了，开始执行
            request->process();
          }
    }
}
#endif
```

说明：

-   构造函数创建所需要的线程数
-   一个线程对应一个任务，任务随时可能完成，线程则可能休眠，所以任务用队列queue实现（线程数量有限），线程用采用wait机制。
-   任务在不断的添加，有可能大于线程数，处于队首的任务先执行。
-   只有添加任务(append)后，才开启线程condition.notify_one()。
-   wait表示，任务为空时，则线程休眠，等待新任务的加入。
-   添加任务时需要添加锁，因为共享资源。

测试代码：

```text
#include "mythread.h"
#include<string>
#include<math.h>
using namespace std;
class Task
{
public:
    void process()
{
        //cout << "run........." << endl;//测试任务数量
        long i=1000000;
        while(i!=0)
        {
            int j = sqrt(i);
            i--;
        }
    }
};
int main(void){
    threadPool<Task> pool(6);//6个线程，vector
    std::string str;
    while (1)
    {
        Task *tt = new Task();//使用智能指针
        pool.append(tt);//不停的添加任务，任务是队列queue，因为只有固定的线程数
        cout<<"添加的任务数量："<<pool.tasks_queue.size()<<endl;  
        delete tt;
    }
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM3MDY5NzgyNywyMTM2MTAzNzE1LDU1Nj
gyOTk1MCw1MTI5Nzg5MDYsLTEyNDQ5Mjk3MzgsNTE0NTczMzQ4
LC03MDIzNTMwOTQsLTU1MjcxMDg2MiwtMjEwNTQ2OTY4Ml19
-->