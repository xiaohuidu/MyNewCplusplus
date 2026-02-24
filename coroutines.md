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
