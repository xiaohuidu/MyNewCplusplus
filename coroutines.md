# 什么是协程？
协程是一种可以暂停执行并在稍后恢复的函数。与普通函数不同，协程可以在执行过程中多次返回控制权，并保持其状态以便后续恢复。

C++20 协程的核心概念
## 1. 协程的三种类型
C++20 引入了三个关键字来支持协程：
- co_await - 暂停执行，等待某个操作完成
- co_yield - 暂停执行并产生一个值（用于生成器）
- co_return - 完成协程执行并返回最终值
只要函数体中包含这三个关键字之一，该函数就是协程。

## 2. 协程的基本结构
```
// 一个简单的协程示例
Generator<int> counter() {
    for (int i = 0; i < 10; ++i) {
        co_yield i;  // 产生值并暂停
    }
}

// 使用协程
auto gen = counter();
for (auto val : gen) {
    std::cout << val << std::endl;
}
```

## 3. 协程的底层机制
协程涉及几个关键组件：
### Promise 类型
Promise 对象控制协程的行为，定义了协程如何启动、暂停和返回值。
```
struct promise_type {
    auto get_return_object() { /* 返回协程句柄 */ }
    auto initial_suspend() { /* 初始暂停点 */ }
    auto final_suspend() noexcept { /* 最终暂停点 */ }
    void return_void() { /* 处理 co_return */ }
    void unhandled_exception() { /* 处理异常 */ }
};
```

### 协程句柄 (Coroutine Handle)
std::coroutine_handle<T> 用于控制协程的执行：
```
std::coroutine_handle<promise_type> handle;
handle.resume();   // 恢复协程执行
handle.destroy();  // 销毁协程
handle.done();     // 检查协程是否完成
```
### Awaitable 对象
与 co_await 一起使用的对象，定义了等待操作的行为：
```
struct awaitable {
    bool await_ready() { /* 是否需要暂停 */ }
    void await_suspend(std::coroutine_handle<>) { /* 暂停时的操作 */ }
    auto await_resume() { /* 恢复时返回的值 */ }
};
```

# 协程的应用场景
## 1. 生成器 (Generators)
```
Generator<int> fibonacci() {
    int a = 0, b = 1;
    while (true) {
        co_yield a;
        auto next = a + b;
        a = b;
        b = next;
    }
}
```
## 2. 异步操作
```
Task<std::string> fetch_data(std::string url) {
    auto response = co_await http_client.get(url);
    auto data = co_await response.read_body();
    co_return data;
}
```
## 3. 惰性求值
```
Generator<int> lazy_range(int start, int end) {
    for (int i = start; i < end; ++i) {
        co_yield i;  // 只在需要时计算
    }
}
```
# 协程的执行流程
```
Task my_coroutine() {
    std::cout << "1. 开始执行\n";
    
    co_await some_async_operation();  // 暂停点
    std::cout << "2. 恢复执行\n";
    
    co_await another_operation();     // 另一个暂停点
    std::cout << "3. 再次恢复\n";
    
    co_return;  // 完成
}
```
执行流程：
- 1. 调用协程时，创建协程帧（coroutine frame）
- 2. 执行到 co_await，保存状态并暂停
- 3. 控制权返回给调用者
- 4. 稍后调用 resume() 恢复执行
- 5. 从暂停点继续执行
- 6. 重复步骤 2-5，直到 co_return
     
# 协程的优势
- 更清晰的异步代码 - 避免回调地狱
- 状态保持 - 自动保存局部变量
- 零开销抽象 - 编译器优化后性能接近手写状态机
- 可组合性 - 协程可以相互调用和组合
# 协程的注意事项
- 生命周期管理 - 需要确保协程句柄被正确销毁
- 悬空引用 - 协程暂停时，引用的对象可能已被销毁
- 异常处理 - 需要在 promise 类型中正确处理异常
- 编译器支持 - 需要 C++20 及支持协程的编译器
# 实际示例：简单的任务调度器
```
struct Task {
    struct promise_type {
        Task get_return_object() {
            return Task{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };
    
    std::coroutine_handle<promise_type> handle;
    
    void resume() { handle.resume(); }
    bool done() { return handle.done(); }
    ~Task() { if (handle) handle.destroy(); }
};

Task example_task() {
    std::cout << "任务开始\n";
    co_await std::suspend_always{};
    std::cout << "任务恢复\n";
    co_await std::suspend_always{};
    std::cout << "任务完成\n";
}
```
# 总结
C++ 协程是一个强大的特性，它提供了：
- 编写异步代码的新方式
- 实现生成器和惰性求值的能力
- 更好的代码可读性和可维护性
- 但它也需要理解底层机制（Promise、Handle、Awaitable）才能有效使用。对于复杂场景，建议使用现有的协程库（如 cppcoro）来简化开发。

# 完整协程示例:
## 1. 基础任务类型(Task)
```
#include <coroutine>
#include <iostream>
#include <exception>
#include <optional>

// 基础的 Task 类型
template<typename T>
class Task {
public:
    struct promise_type {
        std::optional<T> result;
        std::exception_ptr exception;

        Task get_return_object() {
            return Task{std::coroutine_handle<promise_type>::from_promise(*this)};
        }

        std::suspend_always initial_suspend() noexcept { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }

        void return_value(T value) {
            result = std::move(value);
        }

        void unhandled_exception() {
            exception = std::current_exception();
        }
    };

    std::coroutine_handle<promise_type> handle;

    explicit Task(std::coroutine_handle<promise_type> h) : handle(h) {}
    
    Task(Task&& other) noexcept : handle(other.handle) {
        other.handle = nullptr;
    }

    ~Task() {
        if (handle) handle.destroy();
    }

    // 禁止拷贝
    Task(const Task&) = delete;
    Task& operator=(const Task&) = delete;

    T get() {
        if (!handle.done()) {
            handle.resume();
        }
        
        auto& promise = handle.promise();
        if (promise.exception) {
            std::rethrow_exception(promise.exception);
        }
        
        return std::move(*promise.result);
    }

    bool done() const {
        return handle.done();
    }

    void resume() {
        if (!handle.done()) {
            handle.resume();
        }
    }
};

// void 特化版本
template<>
class Task<void> {
public:
    struct promise_type {
        std::exception_ptr exception;

        Task get_return_object() {
            return Task{std::coroutine_handle<promise_type>::from_promise(*this)};
        }

        std::suspend_always initial_suspend() noexcept { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }

        void return_void() {}

        void unhandled_exception() {
            exception = std::current_exception();
        }
    };

    std::coroutine_handle<promise_type> handle;

    explicit Task(std::coroutine_handle<promise_type> h) : handle(h) {}
    
    Task(Task&& other) noexcept : handle(other.handle) {
        other.handle = nullptr;
    }

    ~Task() {
        if (handle) handle.destroy();
    }

    Task(const Task&) = delete;
    Task& operator=(const Task&) = delete;

    void get() {
        while (!handle.done()) {
            handle.resume();
        }
        
        if (handle.promise().exception) {
            std::rethrow_exception(handle.promise().exception);
        }
    }

    void resume() {
        if (!handle.done()) {
            handle.resume();
        }
    }
};
```
## 2. 生成器(Generator)
```
#include <coroutine>
#include <optional>

template<typename T>
class Generator {
public:
    struct promise_type {
        std::optional<T> current_value;

        Generator get_return_object() {
            return Generator{std::coroutine_handle<promise_type>::from_promise(*this)};
        }

        std::suspend_always initial_suspend() noexcept { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }

        std::suspend_always yield_value(T value) {
            current_value = std::move(value);
            return {};
        }

        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    std::coroutine_handle<promise_type> handle;

    explicit Generator(std::coroutine_handle<promise_type> h) : handle(h) {}
    
    Generator(Generator&& other) noexcept : handle(other.handle) {
        other.handle = nullptr;
    }

    ~Generator() {
        if (handle) handle.destroy();
    }

    Generator(const Generator&) = delete;
    Generator& operator=(const Generator&) = delete;

    // 迭代器支持
    class iterator {
        std::coroutine_handle<promise_type> handle;

    public:
        using iterator_category = std::input_iterator_tag;
        using value_type = T;
        using difference_type = std::ptrdiff_t;
        using pointer = T*;
        using reference = T&;

        explicit iterator(std::coroutine_handle<promise_type> h) : handle(h) {}

        iterator& operator++() {
            handle.resume();
            if (handle.done()) {
                handle = nullptr;
            }
            return *this;
        }

        bool operator==(const iterator& other) const {
            return handle == other.handle;
        }

        bool operator!=(const iterator& other) const {
            return !(*this == other);
        }

        const T& operator*() const {
            return *handle.promise().current_value;
        }
    };

    iterator begin() {
        if (handle) {
            handle.resume();
            if (handle.done()) {
                return iterator{nullptr};
            }
        }
        return iterator{handle};
    }

    iterator end() {
        return iterator{nullptr};
    }
};

// 使用示例：斐波那契数列生成器
Generator<int> fibonacci(int count) {
    int a = 0, b = 1;
    for (int i = 0; i < count; ++i) {
        co_yield a;
        auto next = a + b;
        a = b;
        b = next;
    }
}

// 使用示例：范围生成器
Generator<int> range(int start, int end, int step = 1) {
    for (int i = start; i < end; i += step) {
        co_yield i;
    }
}

// 使用示例：文件行读取器
Generator<std::string> read_lines(const std::string& filename) {
    std::ifstream file(filename);
    std::string line;
    while (std::getline(file, line)) {
        co_yield line;
    }
}
```

## 3. 异步操作示例
```
#include <thread>
#include <chrono>
#include <queue>
#include <functional>

// 简单的异步等待器
struct AsyncAwaiter {
    std::chrono::milliseconds duration;
    
    bool await_ready() const noexcept {
        return duration.count() == 0;
    }
    
    void await_suspend(std::coroutine_handle<> handle) {
        // 在新线程中等待后恢复协程
        std::thread([handle, duration = this->duration]() {
            std::this_thread::sleep_for(duration);
            handle.resume();
        }).detach();
    }
    
    void await_resume() noexcept {}
};

// 异步延迟函数
AsyncAwaiter async_sleep(std::chrono::milliseconds ms) {
    return AsyncAwaiter{ms};
}

// 模拟异步 HTTP 请求
struct AsyncHttpRequest {
    std::string url;
    std::string response;
    
    bool await_ready() const noexcept { return false; }
    
    void await_suspend(std::coroutine_handle<> handle) {
        // 模拟异步 HTTP 请求
        std::thread([this, handle]() {
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
            response = "Response from " + url;
            handle.resume();
        }).detach();
    }
    
    std::string await_resume() noexcept {
        return std::move(response);
    }
};

AsyncHttpRequest fetch(const std::string& url) {
    return AsyncHttpRequest{url};
}

// 使用异步操作的协程
Task<std::string> fetch_user_data(int user_id) {
    std::cout << "Fetching user " << user_id << "...\n";
    
    // 异步获取用户信息
    auto user_info = co_await fetch("https://api.example.com/users/" + 
                                    std::to_string(user_id));
    
    std::cout << "Got user info, fetching posts...\n";
    
    // 异步获取用户帖子
    auto posts = co_await fetch("https://api.example.com/users/" + 
                                std::to_string(user_id) + "/posts");
    
    co_return user_info + " | " + posts;
}
```
## 4. 任务调度器
```
#include <queue>
#include <mutex>
#include <condition_variable>

class Scheduler {
    std::queue<std::coroutine_handle<>> tasks;
    std::mutex mutex;
    std::condition_variable cv;
    bool stopped = false;

public:
    void schedule(std::coroutine_handle<> handle) {
        {
            std::lock_guard<std::mutex> lock(mutex);
            tasks.push(handle);
        }
        cv.notify_one();
    }

    void run() {
        while (true) {
            std::coroutine_handle<> handle;
            
            {
                std::unique_lock<std::mutex> lock(mutex);
                cv.wait(lock, [this] { return !tasks.empty() || stopped; });
                
                if (stopped && tasks.empty()) {
                    break;
                }
                
                handle = tasks.front();
                tasks.pop();
            }
            
            if (handle && !handle.done()) {
                handle.resume();
            }
        }
    }

    void stop() {
        {
            std::lock_guard<std::mutex> lock(mutex);
            stopped = true;
        }
        cv.notify_all();
    }
};

// 全局调度器
inline Scheduler global_scheduler;

// 可调度的等待器
struct ScheduledAwaiter {
    bool await_ready() const noexcept { return false; }
    
    void await_suspend(std::coroutine_handle<> handle) {
        global_scheduler.schedule(handle);
    }
    
    void await_resume() noexcept {}
};

ScheduledAwaiter yield() {
    return {};
}
```
## 5. 完整的应用示例
```
#include <iostream>
#include <vector>
#include <string>

// 示例 1：使用生成器
void example_generator() {
    std::cout << "=== Generator Example ===\n";
    
    // 斐波那契数列
    std::cout << "Fibonacci: ";
    for (auto num : fibonacci(10)) {
        std::cout << num << " ";
    }
    std::cout << "\n";
    
    // 范围
    std::cout << "Range: ";
    for (auto num : range(0, 20, 2)) {
        std::cout << num << " ";
    }
    std::cout << "\n\n";
}

// 示例 2：异步任务链
Task<void> async_task_chain() {
    std::cout << "=== Async Task Chain ===\n";
    
    std::cout << "Starting task 1...\n";
    co_await async_sleep(std::chrono::milliseconds(500));
    std::cout << "Task 1 completed\n";
    
    std::cout << "Starting task 2...\n";
    co_await async_sleep(std::chrono::milliseconds(500));
    std::cout << "Task 2 completed\n";
    
    std::cout << "Starting task 3...\n";
    co_await async_sleep(std::chrono::milliseconds(500));
    std::cout << "Task 3 completed\n\n";
}

// 示例 3：并发任务
Task<int> compute_async(int x, int delay_ms) {
    std::cout << "Computing " << x << "...\n";
    co_await async_sleep(std::chrono::milliseconds(delay_ms));
    co_return x * x;
}

Task<void> parallel_tasks() {
    std::cout << "=== Parallel Tasks ===\n";
    
    // 启动多个任务
    auto task1 = compute_async(5, 300);
    auto task2 = compute_async(7, 200);
    auto task3 = compute_async(9, 100);
    
    // 等待所有任务完成
    int result1 = task1.get();
    int result2 = task2.get();
    int result3 = task3.get();
    
    std::cout << "Results: " << result1 << ", " 
              << result2 << ", " << result3 << "\n\n";
}

// 示例 4：错误处理
Task<int> may_fail(bool should_fail) {
    std::cout << "Task starting...\n";
    co_await async_sleep(std::chrono::milliseconds(100));
    
    if (should_fail) {
        throw std::runtime_error("Task failed!");
    }
    
    co_return 42;
}

Task<void> error_handling_example() {
    std::cout << "=== Error Handling ===\n";
    
    try {
        auto result = may_fail(false).get();
        std::cout << "Success: " << result << "\n";
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << "\n";
    }
    
    try {
        auto result = may_fail(true).get();
        std::cout << "Success: " << result << "\n";
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << "\n";
    }
    
    std::cout << "\n";
}

// 示例 5：数据处理流水线
Generator<int> filter_even(Generator<int> gen) {
    for (auto num : gen) {
        if (num % 2 == 0) {
            co_yield num;
        }
    }
}

Generator<int> transform_double(Generator<int> gen) {
    for (auto num : gen) {
        co_yield num * 2;
    }
}

void pipeline_example() {
    std::cout << "=== Pipeline Example ===\n";
    
    auto numbers = range(1, 11);
    auto evens = filter_even(std::move(numbers));
    auto doubled = transform_double(std::move(evens));
    
    std::cout << "Even numbers doubled: ";
    for (auto num : doubled) {
        std::cout << num << " ";
    }
    std::cout << "\n\n";
}

// 主函数
int main() {
    // 生成器示例
    example_generator();
    
    // 异步任务链
    auto chain_task = async_task_chain();
    chain_task.get();
    
    // 并发任务
    auto parallel = parallel_tasks();
    parallel.get();
    
    // 错误处理
    auto error_task = error_handling_example();
    error_task.get();
    
    // 数据流水线
    pipeline_example();
    
    std::cout << "All examples completed!\n";
    
    return 0;
}
```
## 6. 实际应用： 简单的Web服务器处理
```
// 模拟 HTTP 请求处理
struct Request {
    std::string method;
    std::string path;
    std::map<std::string, std::string> headers;
    std::string body;
};

struct Response {
    int status_code;
    std::string body;
};

// 异步数据库查询
Task<std::string> query_database(const std::string& query) {
    std::cout << "Querying database: " << query << "\n";
    co_await async_sleep(std::chrono::milliseconds(50));
    co_return "Database result for: " + query;
}

// 异步缓存查询
Task<std::optional<std::string>> check_cache(const std::string& key) {
    std::cout << "Checking cache: " << key << "\n";
    co_await async_sleep(std::chrono::milliseconds(10));
    
    // 模拟缓存未命中
    co_return std::nullopt;
}

// HTTP 请求处理器
Task<Response> handle_request(const Request& req) {
    std::cout << "Handling " << req.method << " " << req.path << "\n";
    
    // 1. 检查缓存
    auto cached = co_await check_cache(req.path);
    if (cached) {
        co_return Response{200, *cached};
    }
    
    // 2. 查询数据库
    auto db_result = co_await query_database(
        "SELECT * FROM pages WHERE path = '" + req.path + "'"
    );
    
    // 3. 处理业务逻辑
    co_await async_sleep(std::chrono::milliseconds(20));
    
    // 4. 返回响应
    co_return Response{200, "Processed: " + db_result};
}

// 使用示例
void web_server_example() {
    std::cout << "=== Web Server Example ===\n";
    
    Request req{
        .method = "GET",
        .path = "/api/users/123",
        .headers = {},
        .body = ""
    };
    
    auto response_task = handle_request(req);
    auto response = response_task.get();
    
    std::cout << "Response: " << response.status_code 
              << " - " << response.body << "\n\n";
}
```
编译和运行
```
# 编译（需要 C++20 支持）
g++ -std=c++20 -fcoroutines -O2 coroutine_example.cpp -o coroutine_example -lpthread

# 或使用 Clang
clang++ -std=c++20 -stdlib=libc++ -O2 coroutine_example.cpp -o coroutine_example -lpthread

# 运行
./coroutine_example
```
输出示例：
```
=== Generator Example ===
Fibonacci: 0 1 1 2 3 5 8 13 21 34 
Range: 0 2 4 6 8 10 12 14 16 18 

=== Async Task Chain ===
Starting task 1...
Task 1 completed
Starting task 2...
Task 2 completed
Starting task 3...
Task 3 completed

=== Parallel Tasks ===
Computing 5...
Computing 7...
Computing 9...
Results: 25, 49, 81

=== Error Handling ===
Task starting...
Success: 42
Task starting...
Caught: Task failed!

=== Pipeline Example ===
Even numbers doubled: 4 8 12 16 20 

All examples completed!
```
这个完整示例展示了：
✅ 基础 Task 类型实现
✅ Generator 生成器模式
✅ 异步操作和等待
✅ 任务调度
✅ 错误处理
✅ 并发任务
✅ 数据流水线
✅ 实际应用场景（Web 服务器）
您可以根据需要修改和扩展这些示例！




