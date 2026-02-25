# C++11/14/17/20 现代特性详解

本文档详细介绍 C++11 及之后版本引入的重要新特性，每个特性都配有代码示例。

---

## 目录

- [C++11 核心特性](#c11-核心特性)
- [C++14 新特性](#c14-新特性)
- [C++17 新特性](#c17-新特性)
- [C++20 新特性](#c20-新特性)

---

## C++11 核心特性

### 1. auto 类型推导

`auto` 关键字让编译器自动推导变量类型，简化代码并提高可读性。

```cpp
#include <iostream>
#include <vector>
#include <map>

void auto_example() {
    // 基本类型推导
    auto x = 42;              // int
    auto y = 3.14;            // double
    auto z = "hello";         // const char*
    auto vec = std::vector<int>{1, 2, 3};
    
    // 与迭代器配合使用（避免冗长的类型声明）
    std::map<std::string, int> m = {{"one", 1}, {"two", 2}};
    
    // 传统写法
    for (std::map<std::string, int>::iterator it = m.begin(); 
         it != m.end(); ++it) {
        std::cout << it->first << ": " << it->second << "\n";
    }
    
    // auto 简化写法
    for (auto it = m.begin(); it != m.end(); ++it) {
        std::cout << it->first << ": " << it->second << "\n";
    }
    
    // 注意：auto 会丢失 const 和引用
    const int ci = 10;
    auto a1 = ci;        // a1 是 int，不是 const int
    const auto a2 = ci;  // a2 是 const int
    auto& a3 = ci;       // a3 是 const int&
}
```

**使用场景：**
- 复杂的模板类型
- 迭代器
- Lambda 表达式
- 函数返回值类型复杂时

---

### 2. decltype 类型推导

`decltype` 用于推导表达式的类型，保留 const 和引用属性。

```cpp
#include <iostream>
#include <vector>

void decltype_example() {
    int x = 42;
    decltype(x) y = 10;  // y 的类型与 x 相同，为 int
    
    const int& cx = x;
    decltype(cx) cy = x;  // cy 是 const int&，保留引用和 const
    
    // 推导表达式的类型
    decltype(x + y) z = x + y;  // z 类型为 int
    
    // 与 auto 的区别
    auto a1 = cx;        // a1 是 int（丢失 const 和引用）
    decltype(cx) a2 = x; // a2 是 const int&（保留）
    
    // 用于函数返回类型推导
    auto func1 = [](int a, int b) -> decltype(a + b) {
        return a + b;
    };
    
    std::cout << func1(3, 4) << "\n";  // 7
}

// C++14: decltype(auto) 结合两者优点
template<typename Container, typename Index>
decltype(auto) get_element(Container& c, Index i) {
    return c[i];  // 保留返回值的引用属性
}
```

**使用场景：**
- 需要保留 const 和引用属性
- 模板编程中推导返回类型
- 与 auto 配合使用

---

### 3. nullptr

类型安全的空指针常量，替代 NULL 和 0。

```cpp
#include <iostream>

void func(int x) { 
    std::cout << "Called func(int): " << x << "\n"; 
}

void func(char* p) { 
    std::cout << "Called func(char*)\n"; 
}

void nullptr_example() {
    // C++11 之前的问题
    // NULL 通常定义为 0，是整数类型
    // func(NULL);  // 歧义！可能调用 func(int)
    
    // C++11 使用 nullptr
    func(nullptr);  // 明确调用 func(char*)
    
    int* ptr1 = nullptr;
    char* ptr2 = nullptr;
    
    // nullptr 可以转换为任何指针类型
    if (ptr1 == nullptr) {
        std::cout << "ptr1 is null\n";
    }
    
    // 类型安全
    // int x = nullptr;  // 错误：不能将 nullptr 赋值给非指针类型
}
```

**优势：**
- 类型安全，避免歧义
- 不能隐式转换为整数
- 可以转换为任何指针类型

---

### 4. 范围 for 循环（Range-based for loop）

简化容器遍历的语法。

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <string>

void range_for_example() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    
    // 基本用法（值拷贝）
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    
    // 使用引用避免拷贝（推荐）
    for (const auto& x : vec) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    
    // 修改元素（需要非 const 引用）
    for (auto& x : vec) {
        x *= 2;
    }
    
    // 遍历 map
    std::map<std::string, int> m = {{"one", 1}, {"two", 2}, {"three", 3}};
    for (const auto& pair : m) {
        std::cout << pair.first << ": " << pair.second << "\n";
    }
    
    // C++17 结构化绑定（更优雅）
    for (const auto& [key, value] : m) {
        std::cout << key << ": " << value << "\n";
    }
    
    // 直接遍历初始化列表
    for (int x : {10, 20, 30, 40, 50}) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    
    // 遍历 C 风格数组
    int arr[] = {1, 2, 3, 4, 5};
    for (int x : arr) {
        std::cout << x << " ";
    }
    std::cout << "\n";
}
```

**注意事项：**
- 使用 `const auto&` 避免不必要的拷贝
- 需要修改元素时使用 `auto&`
- 不能在循环中修改容器大小

---

### 5. Lambda 表达式

匿名函数对象，简化函数式编程。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>

void lambda_example() {
    // 基本语法：[捕获列表](参数列表) -> 返回类型 { 函数体 }
    
    // 1. 最简单的 lambda
    auto hello = []() { std::cout << "Hello Lambda!\n"; };
    hello();
    
    // 2. 带参数和返回值
    auto add = [](int a, int b) -> int { return a + b; };
    std::cout << "3 + 4 = " << add(3, 4) << "\n";
    
    // 3. 返回类型自动推导（可省略 -> int）
    auto multiply = [](int a, int b) { return a * b; };
    std::cout << "3 * 4 = " << multiply(3, 4) << "\n";
    
    // 4. 捕获外部变量
    int x = 10;
    int y = 20;
    
    // 值捕获（拷贝）
    auto lambda1 = [x]() { return x * 2; };
    std::cout << "lambda1: " << lambda1() << "\n";  // 20
    
    // 引用捕获（可修改外部变量）
    auto lambda2 = [&x]() { x *= 2; };
    lambda2();
    std::cout << "x after lambda2: " << x << "\n";  // 20
    
    // 捕获所有外部变量（值）
    auto lambda3 = [=]() { return x + y; };
    std::cout << "lambda3: " << lambda3() << "\n";  // 40
    
    // 捕获所有外部变量（引用）
    auto lambda4 = [&]() { x++; y++; };
    lambda4();
    std::cout << "x=" << x << ", y=" << y << "\n";  // 21, 21
    
    // 混合捕获
    auto lambda5 = [x, &y]() { 
        // x 是值捕获，y 是引用捕获
        y = x + y;
    };
    lambda5();
    std::cout << "y after lambda5: " << y << "\n";  // 41
    
    // 5. mutable lambda（允许修改值捕获的变量）
    int count = 0;
    auto counter = [count]() mutable { return ++count; };
    std::cout << counter() << "\n";  // 1
    std::cout << counter() << "\n";  // 2
    std::cout << "Original count: " << count << "\n";  // 0（外部变量未改变）
    
    // 6. 与 STL 算法配合
    std::vector<int> vec = {5, 2, 8, 1, 9, 3};
    
    // 排序（降序）
    std::sort(vec.begin(), vec.end(), [](int a, int b) {
        return a > b;
    });
    
    // 查找大于 5 的元素
    auto it = std::find_if(vec.begin(), vec.end(), [](int x) {
        return x > 5;
    });
    
    if (it != vec.end()) {
        std::cout << "Found: " << *it << "\n";
    }
    
    // 统计偶数个数
    int even_count = std::count_if(vec.begin(), vec.end(), [](int x) {
        return x % 2 == 0;
    });
    std::cout << "Even numbers: " << even_count << "\n";
    
    // 7. 存储 lambda（使用 std::function）
    std::function<int(int, int)> operation;
    operation = [](int a, int b) { return a + b; };
    std::cout << "Operation: " << operation(10, 20) << "\n";
}
```

**捕获列表说明：**
- `[]`：不捕获任何变量
- `[=]`：值捕获所有外部变量
- `[&]`：引用捕获所有外部变量
- `[x]`：值捕获 x
- `[&x]`：引用捕获 x
- `[x, &y]`：x 值捕获，y 引用捕获
- `[this]`：捕获当前对象指针

---

### 6. 智能指针

自动管理内存，避免内存泄漏。

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Resource {
public:
    Resource(int id) : id_(id) {
        std::cout << "Resource " << id_ << " created\n";
    }
    
    ~Resource() {
        std::cout << "Resource " << id_ << " destroyed\n";
    }
    
    void use() {
        std::cout << "Using resource " << id_ << "\n";
    }
    
private:
    int id_;
};

void smart_pointer_example() {
    std::cout << "=== unique_ptr 示例 ===\n";
    {
        // 1. unique_ptr：独占所有权
        std::unique_ptr<Resource> ptr1(new Resource(1));
        
        // C++14 推荐：使用 make_unique（更安全）
        auto ptr2 = std::make_unique<Resource>(2);
        
        ptr1->use();
        ptr2->use();
        
        // 不能拷贝
        // std::unique_ptr<Resource> ptr3 = ptr1;  // 错误！
        
        // 只能移动
        std::unique_ptr<Resource> ptr3 = std::move(ptr1);
        // ptr1 现在为空
        if (!ptr1) {
            std::cout << "ptr1 is now null\n";
        }
        ptr3->use();
        
        // 数组版本
        auto arr = std::make_unique<int[]>(10);
        for (int i = 0; i < 10; ++i) {
            arr[i] = i;
        }
        
        // 自定义删除器
        auto deleter = [](Resource* p) {
            std::cout << "Custom deleter called\n";
            delete p;
        };
        std::unique_ptr<Resource, decltype(deleter)> ptr4(
            new Resource(4), deleter
        );
        
    }  // ptr2, ptr3, ptr4 自动销毁
    
    std::cout << "\n=== shared_ptr 示例 ===\n";
    {
        // 2. shared_ptr：共享所有权（引用计数）
        std::shared_ptr<Resource> sptr1 = std::make_shared<Resource>(10);
        std::cout << "Use count: " << sptr1.use_count() << "\n";  // 1
        
        {
            std::shared_ptr<Resource> sptr2 = sptr1;  // 可以拷贝
            std::cout << "Use count: " << sptr1.use_count() << "\n";  // 2
            
            std::shared_ptr<Resource> sptr3 = sptr1;
            std::cout << "Use count: " << sptr1.use_count() << "\n";  // 3
            
            sptr2->use();
        }  // sptr2 和 sptr3 离开作用域，引用计数 -2
        
        std::cout << "Use count: " << sptr1.use_count() << "\n";  // 1
    }  // sptr1 离开作用域，资源被销毁
    
    std::cout << "\n=== weak_ptr 示例 ===\n";
    {
        // 3. weak_ptr：弱引用，不增加引用计数
        std::shared_ptr<Resource> sptr = std::make_shared<Resource>(20);
        std::weak_ptr<Resource> wptr = sptr;
        
        std::cout << "Use count: " << sptr.use_count() << "\n";  // 1（weak_ptr 不增加）
        
        // 使用 weak_ptr（需要先转换为 shared_ptr）
        if (auto locked = wptr.lock()) {  // 尝试获取 shared_ptr
            locked->use();
            std::cout << "Use count: " << sptr.use_count() << "\n";  // 2
        }
        
        // 重置 shared_ptr
        sptr.reset();
        
        // weak_ptr 检测对象是否还存在
        if (wptr.expired()) {
            std::cout << "Resource has been destroyed\n";
        }
    }
    
    std::cout << "\n=== 循环引用问题 ===\n";
    {
        struct Node {
            std::shared_ptr<Node> next;
            std::weak_ptr<Node> prev;  // 使用 weak_ptr 打破循环引用
            int data;
            
            Node(int d) : data(d) {
                std::cout << "Node " << data << " created\n";
            }
            
            ~Node() {
                std::cout << "Node " << data << " destroyed\n";
            }
        };
        
        auto node1 = std::make_shared<Node>(1);
        auto node2 = std::make_shared<Node>(2);
        
        node1->next = node2;
        node2->prev = node1;  // 使用 weak_ptr，不会造成循环引用
        
    }  // 两个节点都会被正确销毁
}
```

**使用建议：**
- 默认使用 `unique_ptr`
- 需要共享所有权时使用 `shared_ptr`
- 打破循环引用时使用 `weak_ptr`
- 优先使用 `make_unique` 和 `make_shared`

---

### 7. 右值引用和移动语义

提高性能，避免不必要的拷贝。

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <utility>

class MyString {
private:
    char* data_;
    size_t size_;
    
public:
    // 构造函数
    MyString(const char* str = "") {
        size_ = std::strlen(str);
        data_ = new char[size_ + 1];
        std::strcpy(data_, str);
        std::cout << "Constructor: " << data_ << "\n";
    }
    
    // 拷贝构造函数
    MyString(const MyString& other) {
        size_ = other.size_;
        data_ = new char[size_ + 1];
        std::strcpy(data_, other.data_);
        std::cout << "Copy constructor: " << data_ << "\n";
    }
    
    // 移动构造函数（C++11）
    MyString(MyString&& other) noexcept {
        data_ = other.data_;
        size_ = other.size_;
        other.data_ = nullptr;
        other.size_ = 0;
        std::cout << "Move constructor\n";
    }
    
    // 拷贝赋值运算符
    MyString& operator=(const MyString& other) {
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            data_ = new char[size_ + 1];
            std::strcpy(data_, other.data_);
            std::cout << "Copy assignment: " << data_ << "\n";
        }
        return *this;
    }
    
    // 移动赋值运算符（C++11）
    MyString& operator=(MyString&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            size_ = other.size_;
            other.data_ = nullptr;
            other.size_ = 0;
            std::cout << "Move assignment\n";
        }
        return *this;
    }
    
    ~MyString() {
        if (data_) {
            std::cout << "Destructor: " << data_ << "\n";
        }
        delete[] data_;
    }
    
    const char* c_str() const { return data_; }
};

void move_semantics_example() {
    std::cout << "=== 左值 vs 右值 ===\n";
    int x = 10;      // x 是左值
    int& lref = x;   // 左值引用
    // int& lref2 = 20;  // 错误：不能绑定到右值
    
    int&& rref = 20;  // 右值引用（绑定到临时对象）
    // int&& rref2 = x;  // 错误：不能绑定到左值
    
    std::cout << "\n=== 移动语义示例 ===\n";
    {
        MyString s1("Hello");
        MyString s2 = s1;              // 拷贝构造
        MyString s3 = std::move(s1);   // 移动构造（s1 被移动后不再有效）
        
        MyString s4("World");
        s4 = s2;                       // 拷贝赋值
        s4 = std::move(s3);            // 移动赋值
    }
    
    std::cout << "\n=== STL 容器中的移动 ===\n";
    {
        std::vector<MyString> vec;
        
        MyString s1("First");
        vec.push_back(s1);              // 拷贝
        vec.push_back(std::move(s1));   // 移动（s1 被移动）
        vec.push_back(MyString("Third"));  // 直接移动临时对象
        
        // emplace_back：直接在容器中构造对象（更高效）
        vec.emplace_back("Fourth");
    }
    
    std::cout << "\n=== 完美转发 ===\n";
    {
        auto process = [](MyString& s) {
            std::cout << "Lvalue version\n";
        };
        
        auto process_rvalue = [](MyString&& s) {
            std::cout << "Rvalue version\n";
        };
        
        // 通用引用 + std::forward 实现完美转发
        auto wrapper = [&](auto&& arg) {
            if constexpr (std::is_lvalue_reference_v<decltype(arg)>) {
                process(arg);
            } else {
                process_rvalue(std::forward<decltype(arg)>(arg));
            }
        };
        
        MyString s("Test");
        wrapper(s);                    // 左值
        wrapper(MyString("Temp"));     // 右值
    }
}
```

**关键概念：**
- **左值**：有名字的对象，可以取地址
- **右值**：临时对象，不能取地址
- **std::move**：将左值转换为右值引用
- **std::forward**：完美转发，保持参数的值类别

---

### 8. 列表初始化（统一初始化）

统一的初始化语法，防止窄化转换。

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <string>

class Point {
public:
    int x, y;
    
    Point(int x, int y) : x(x), y(y) {}
    
    void print() const {
        std::cout << "(" << x << ", " << y << ")\n";
    }
};

void uniform_initialization_example() {
    // 1. 基本类型
    int x{42};
    int y = {42};
    double d{3.14};
    
    // 2. 防止窄化转换
    // int narrow{3.14};  // 错误：窄化转换
    int truncate(3.14);   // 允许，但会截断
    
    // 3. 数组
    int arr1[]{1, 2, 3, 4, 5};
    int arr2[5]{1, 2, 3};  // 剩余元素初始化为 0
    
    // 4. STL 容器
    std::vector<int> vec{1, 2, 3, 4, 5};
    std::map<std::string, int> m{
        {"one", 1},
        {"two", 2},
        {"three", 3}
    };
    
    // 5. 自定义类型
    Point p1{10, 20};
    Point p2 = {30, 40};
    p1.print();
    p2.print();
    
    // 6. 初始化列表的歧义
    std::vector<int> v1{10, 20};      // 2 个元素：10, 20
    std::vector<int> v2(10, 20);      // 10 个元素，每个值为 20
    
    std::cout << "v1 size: " << v1.size() << "\n";  // 2
    std::cout << "v2 size: " << v2.size() << "\n";  // 10
    
    // 7. 返回值
    auto get_point = []() -> Point {
        return {100, 200};  // 统一初始化
    };
    
    auto p3 = get_point();
    p3.print();
    
    // 8. 聚合初始化
    struct Data {
        int a;
        double b;
        std::string c;
    };
    
    Data d1{42, 3.14, "hello"};
    std::cout << d1.a << ", " << d1.b << ", " << d1.c << "\n";
}
```

**优势：**
- 统一的语法
- 防止窄化转换
- 可以用于任何类型
- 更安全

---

### 9. constexpr

编译期常量表达式，提高性能。

```cpp
#include <iostream>
#include <array>

// 编译期计算
constexpr int square(int x) {
    return x * x;
}

constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}

// C++14: 允许更复杂的函数
constexpr int fibonacci(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; ++i) {
        int temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}

// constexpr 类
class Point {
    int x_, y_;
    
public:
    constexpr Point(int x, int y) : x_(x), y_(y) {}
    
    constexpr int x() const { return x_; }
    constexpr int y() const { return y_; }
    
    constexpr int distance_squared() const {
        return x_ * x_ + y_ * y_;
    }
};

void constexpr_example() {
    // 编译期计算
    constexpr int s = square(5);           // 编译期计算
    constexpr int f = factorial(5);        // 编译期计算
    constexpr int fib = fibonacci(10);     // 编译期计算
    
    std::cout << "square(5) = " << s << "\n";
    std::cout << "factorial(5) = " << f << "\n";
    std::cout << "fibonacci(10) = " << fib << "\n";
    
    // 可以用作数组大小（必须是编译期常量）
    int arr[square(3)];  // 数组大小为 9
    std::cout << "Array size: " << sizeof(arr) / sizeof(int) << "\n";
    
    // constexpr 对象
    constexpr Point p(3, 4);
    constexpr int dist = p.distance_squared();  // 编译期计算
    std::cout << "Distance squared: " << dist << "\n";  // 25
    
    // 运行时也可以使用
    int runtime_value = 10;
    int result = square(runtime_value);  // 运行时计算
    std::cout << "Runtime square: " << result << "\n";
    
    // C++17: constexpr if
    auto check_type = [](auto x) {
        if constexpr (std::is_integral_v<decltype(x)>) {
            std::cout << "Integer type\n";
        } else if constexpr (std::is_floating_point_v<decltype(x)>) {
            std::cout << "Floating point type\n";
        } else {
            std::cout << "Other type\n";
        }
    };
    
    check_type(42);
    check_type(3.14);
    check_type("hello");
}
```

**使用场景：**
- 编译期计算，提高运行时性能
- 数组大小、模板参数等需要编译期常量的地方
- 元编程

---

### 10. 变参模板

支持任意数量的模板参数。

```cpp
#include <iostream>
#include <string>

// 1. 递归展开（C++11）
void print() {
    std::cout << "\n";
}

template<typename T, typename... Args>
void print(T first, Args... args) {
    std::cout << first << " ";
    print(args...);  // 递归调用
}

// 2. sizeof... 获取参数包大小
template<typename... Args>
void count_args(Args... args) {
    std::cout << "Number of arguments: " << sizeof...(args) << "\n";
}

// 3. 完美转发
template<typename... Args>
void forward_example(Args&&... args) {
    print(std::forward<Args>(args)...);
}

// 4. 变参模板类
template<typename... Types>
class Tuple;

template<>
class Tuple<> {
public:
    void print() const {
        std::cout << "()\n";
    }
};

template<typename T, typename... Rest>
class Tuple<T, Rest...> : public Tuple<Rest...> {
    T value_;
    
public:
    Tuple(T value, Rest... rest) 
        : Tuple<Rest...>(rest...), value_(value) {}
    
    T value() const { return value_; }
    
    void print() const {
        std::cout << value_ << " ";
        Tuple<Rest...>::print();
    }
};

// 5. 折叠表达式（C++17）
template<typename... Args>
auto sum_fold(Args... args) {
    return (args + ...);  // 一元右折叠
}

template<typename... Args>
auto sum_fold_left(Args... args) {
    return (... + args);  // 一元左折叠
}

template<typename... Args>
void print_fold(Args... args) {
    ((std::cout << args << " "), ...);  // 二元左折叠
    std::cout << "\n";
}

void variadic_template_example() {
    std::cout << "=== 递归展开 ===\n";
    print(1, 2.5, "hello", 'c');
    
    std::cout << "\n=== sizeof... ===\n";
    count_args(1, 2, 3, 4, 5);
    count_args("a", "b", "c");
    
    std::cout << "\n=== 完美转发 ===\n";
    forward_example(1, 2.5, "world");
    
    std::cout << "\n=== 变参模板类 ===\n";
    Tuple<int, double, std::string> t(42, 3.14, "tuple");
    t.print();
    
    std::cout << "\n=== 折叠表达式（C++17）===\n";
    std::cout << "Sum: " << sum_fold(1, 2, 3, 4, 5) << "\n";
    std::cout << "Sum left: " << sum_fold_left(1, 2, 3, 4, 5) << "\n";
    print_fold(1, 2, 3, "hello", 4.5);
}
```

**应用场景：**
- 实现 `printf` 风格的函数
- 元组（tuple）实现
- 完美转发
- 通用工厂函数

---

### 11. std::thread 多线程

C++11 引入标准线程库。

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <vector>
#include <chrono>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;
int shared_data = 0;

// 简单的线程函数
void task(int id) {
    std::cout << "Thread " << id << " is running\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    std::cout << "Thread " << id << " finished\n";
}

// 使用互斥锁保护共享数据
void increment_with_lock(int times) {
    for (int i = 0; i < times; ++i) {
        std::lock_guard<std::mutex> lock(mtx);  // RAII 自动解锁
        ++shared_data;
    }
}

// 使用 unique_lock（更灵活）
void increment_with_unique_lock(int times) {
    for (int i = 0; i < times; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        ++shared_data;
        lock.unlock();  // 可以手动解锁
        
        // 做一些不需要锁的工作
        std::this_thread::sleep_for(std::chrono::microseconds(1));
        
        lock.lock();    // 可以重新加锁
    }
}

// 条件变量示例
void wait_for_signal(int id) {
    std::unique_lock<std::mutex> lock(mtx);
    std::cout << "Thread " << id << " waiting...\n";
    cv.wait(lock, []{ return ready; });  // 等待条件满足
    std::cout << "Thread " << id << " proceeding\n";
}

void send_signal() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
        std::cout << "Signal sent\n";
    }
    cv.notify_all();  // 通知所有等待的线程
}

void thread_example() {
    std::cout << "=== 基本线程创建 ===\n";
    {
        std::thread t1(task, 1);
        std::thread t2([](int id) {
            std::cout << "Lambda thread " << id << "\n";
        }, 2);
        
        t1.join();  // 等待线程完成
        t2.join();
    }
    
    std::cout << "\n=== 互斥锁示例 ===\n";
    {
        shared_data = 0;
        std::vector<std::thread> threads;
        
        for (int i = 0; i < 10; ++i) {
            threads.emplace_back(increment_with_lock, 1000);
        }
        
        for (auto& t : threads) {
            t.join();
        }
        
        std::cout << "Final value: " << shared_data << "\n";  // 应该是 10000
    }
    
    std::cout << "\n=== 条件变量示例 ===\n";
    {
        ready = false;
        std::thread t1(wait_for_signal, 1);
        std::thread t2(wait_for_signal, 2);
        std::thread t3(send_signal);
        
        t1.join();
        t2.join();
        t3.join();
    }
    
    std::cout << "\n=== 线程信息 ===\n";
    {
        std::cout << "Hardware concurrency: " 
                  << std::thread::hardware_concurrency() << "\n";
        
        std::thread t(task, 1);
        std::cout << "Thread ID: " << t.get_id() << "\n";
        t.join();
    }
}
```

**同步原语：**
- `std::mutex`：互斥锁
- `std::lock_guard`：RAII 锁管理
- `std::unique_lock`：更灵活的锁
- `std::condition_variable`：条件变量
- `std::atomic`：原子操作

---

### 12. std::async 和 std::future

异步任务和 future/promise。

```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

// 耗时计算
int compute(int x) {
    std::cout << "Computing " << x << "...\n";
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return x * x;
}

void async_example() {
    std::cout << "=== std::async 示例 ===\n";
    {
        // 启动异步任务
        std::future<int> result = std::async(std::launch::async, compute, 10);
        
        std::cout << "Doing other work...\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
        
        // 获取结果（会阻塞直到任务完成）
        std::cout << "Result: " << result.get() << "\n";
    }
    
    std::cout << "\n=== 多个异步任务 ===\n";
    {
        auto f1 = std::async(std::launch::async, compute, 5);
        auto f2 = std::async(std::launch::async, compute, 7);
        auto f3 = std::async(std::launch::async, compute, 9);
        
        std::cout << "Results: " 
                  << f1.get() << ", " 
                  << f2.get() << ", " 
                  << f3.get() << "\n";
    }
    
    std::cout << "\n=== promise 和 future ===\n";
    {
        std::promise<int> prom;
        std::future<int> fut = prom.get_future();
        
        std::thread t([&prom]() {
            std::this_thread::sleep_for(std::chrono::seconds(1));
            prom.set_value(42);  // 设置值
        });
        
        std::cout << "Waiting for value...\n";
        std::cout << "Value: " << fut.get() << "\n";  // 获取值
        t.join();
    }
    
    std::cout << "\n=== 异常传递 ===\n";
    {
        auto task = []() -> int {
            std::this_thread::sleep_for(std::chrono::milliseconds(500));
            throw std::runtime_error("Task failed!");
            return 0;
        };
        
        std::future<int> result = std::async(std::launch::async, task);
        
        try {
            result.get();
        } catch (const std::exception& e) {
            std::cout << "Caught exception: " << e.what() << "\n";
        }
    }
    
    std::cout << "\n=== 启动策略 ===\n";
    {
        // std::launch::async - 立即在新线程中执行
        auto f1 = std::async(std::launch::async, compute, 3);
        
        // std::launch::deferred - 延迟执行（调用 get 时才执行）
        auto f2 = std::async(std::launch::deferred, compute, 4);
        
        std::cout << "Before get\n";
        std::cout << "Result: " << f2.get() << "\n";  // 此时才执行
        
        f1.get();
    }
}
```

**关键概念：**
- `std::async`：启动异步任务
- `std::future`：获取异步结果
- `std::promise`：设置异步结果
- `std::packaged_task`：打包可调用对象

---

### 13. std::tuple

固定大小的异构容器。

```cpp
#include <iostream>
#include <tuple>
#include <string>

void tuple_example() {
    std::cout << "=== 创建 tuple ===\n";
    {
        // 创建方式
        std::tuple<int, double, std::string> t1(42, 3.14, "hello");
        auto t2 = std::make_tuple(1, 2.5, 'c');
        
        // 访问元素
        std::cout << "t1[0]: " << std::get<0>(t1) << "\n";
        std::cout << "t1[1]: " << std::get<1>(t1) << "\n";
        std::cout << "t1[2]: " << std::get<2>(t1) << "\n";
        
        // 修改元素
        std::get<0>(t1) = 100;
        std::cout << "Modified t1[0]: " << std::get<0>(t1) << "\n";
    }
    
    std::cout << "\n=== 解包 tuple ===\n";
    {
        auto t = std::make_tuple(1, 2.5, "world");
        
        // C++17 结构化绑定（推荐）
        auto [x, y, z] = t;
        std::cout << x << ", " << y << ", " << z << "\n";
        
        // C++11 使用 tie
        int a;
        double b;
        std::string c;
        std::tie(a, b, c) = t;
        std::cout << a << ", " << b << ", " << c << "\n";
        
        // 忽略某些值
        std::tie(a, std::ignore, c) = t;
        std::cout << "Ignored middle: " << a << ", " << c << "\n";
    }
    
    std::cout << "\n=== tuple 操作 ===\n";
    {
        auto t1 = std::make_tuple(1, 2, 3);
        auto t2 = std::make_tuple(4, 5, 6);
        
        // 连接 tuple
        auto t3 = std::tuple_cat(t1, t2);
        std::cout << "Concatenated size: " << std::tuple_size<decltype(t3)>::value << "\n";
        
        // 比较
        std::cout << "t1 < t2: " << (t1 < t2) << "\n";
        
        // 获取类型
        using T = std::tuple_element<1, decltype(t1)>::type;  // int
        std::cout << "Type of t1[1]: " << typeid(T).name() << "\n";
    }
    
    std::cout << "\n=== 实际应用 ===\n";
    {
        // 函数返回多个值
        auto divide = [](int a, int b) -> std::tuple<int, int> {
            return {a / b, a % b};  // 商和余数
        };
        
        auto [quotient, remainder] = divide(17, 5);
        std::cout << "17 / 5 = " << quotient << " remainder " << remainder << "\n";
        
        // 作为 map 的值类型
        std::map<std::string, std::tuple<int, double, std::string>> data;
        data["user1"] = std::make_tuple(25, 175.5, "Alice");
        data["user2"] = std::make_tuple(30, 180.0, "Bob");
        
        for (const auto& [key, value] : data) {
            auto [age, height, name] = value;
            std::cout << key << ": " << name << ", " << age << ", " << height << "\n";
        }
    }
}
```

**使用场景：**
- 函数返回多个值
- 临时打包数据
- 作为容器的值类型

---

### 14. std::array

固定大小的数组容器。

```cpp
#include <iostream>
#include <array>
#include <algorithm>

void array_example() {
    std::cout << "=== 创建和初始化 ===\n";
    {
        // 创建
        std::array<int, 5> arr1 = {1, 2, 3, 4, 5};
        std::array<int, 5> arr2{10, 20, 30};  // 剩余元素为 0
        
        // 大小在编译期确定
        std::cout << "Size: " << arr1.size() << "\n";
        std::cout << "Max size: " << arr1.max_size() << "\n";
        
        // 访问元素
        std::cout << "arr1[0]: " << arr1[0] << "\n";
        std::cout << "arr1.at(1): " << arr1.at(1) << "\n";  // 带边界检查
        std::cout << "arr1.front(): " << arr1.front() << "\n";
        std::cout << "arr1.back(): " << arr1.back() << "\n";
    }
    
    std::cout << "\n=== 迭代 ===\n";
    {
        std::array<int, 5> arr = {1, 2, 3, 4, 5};
        
        // 范围 for
        for (const auto& x : arr) {
            std::cout << x << " ";
        }
        std::cout << "\n";
        
        // 迭代器
        for (auto it = arr.begin(); it != arr.end(); ++it) {
            std::cout << *it << " ";
        }
        std::cout << "\n";
        
        // 反向迭代
        for (auto it = arr.rbegin(); it != arr.rend(); ++it) {
            std::cout << *it << " ";
        }
        std::cout << "\n";
    }
    
    std::cout << "\n=== STL 算法 ===\n";
    {
        std::array<int, 5> arr = {5, 2, 8, 1, 9};
        
        // 排序
        std::sort(arr.begin(), arr.end());
        std::cout << "Sorted: ";
        for (int x : arr) std::cout << x << " ";
        std::cout << "\n";
        
        // 查找
        auto it = std::find(arr.begin(), arr.end(), 8);
        if (it != arr.end()) {
            std::cout << "Found 8 at position " << (it - arr.begin()) << "\n";
        }
        
        // 填充
        arr.fill(0);
        std::cout << "After fill: ";
        for (int x : arr) std::cout << x << " ";
        std::cout << "\n";
    }
    
    std::cout << "\n=== 多维数组 ===\n";
    {
        std::array<std::array<int, 3>, 2> matrix = {{
            {1, 2, 3},
            {4, 5, 6}
        }};
        
        for (const auto& row : matrix) {
            for (int val : row) {
                std::cout << val << " ";
            }
            std::cout << "\n";
        }
    }
    
    std::cout << "\n=== 返回数组 ===\n";
    {
        auto get_array = []() -> std::array<int, 3> {
            return {1, 2, 3};
        };
        
        auto arr = get_array();
        for (int x : arr) std::cout << x << " ";
        std::cout << "\n";
    }
}
```

**优势：**
- 大小固定，性能好
- 支持 STL 算法
- 可以作为函数返回值
- 类型安全

---

## C++14 新特性

### 1. 泛型 Lambda

Lambda 参数可以使用 `auto`。

```cpp
#include <iostream>
#include <vector>
#include <string>

void generic_lambda_example() {
    // 参数类型使用 auto
    auto add = [](auto x, auto y) {
        return x + y;
    };
    
    std::cout << add(1, 2) << "\n";           // int + int = 3
    std::cout << add(1.5, 2.5) << "\n";       // double + double = 4.0
    std::cout << add(std::string("hello"), std::string(" world")) << "\n";  // string + string
    
    // 模板化的 lambda
    auto print = [](const auto& container) {
        for (const auto& item : container) {
            std::cout << item << " ";
        }
        std::cout << "\n";
    };
    
    std::vector<int> vec = {1, 2, 3, 4, 5};
    std::vector<std::string> strs = {"hello", "world"};
    
    print(vec);
    print(strs);
    
    // 完美转发
    auto forward_lambda = [](auto&&... args) {
        return add(std::forward<decltype(args)>(args)...);
    };
    
    std::cout << forward_lambda(10, 20) << "\n";
}
```

---

### 2. Lambda 捕获初始化

在捕获列表中初始化变量。

```cpp
#include <iostream>
#include <memory>

void lambda_init_capture_example() {
    int x = 10;
    
    // 捕获时初始化新变量
    auto lambda1 = [y = x + 1]() {
        return y * 2;
    };
    std::cout << lambda1() << "\n";  // 22
    
    // 移动捕获（unique_ptr 不能拷贝）
    auto ptr = std::make_unique<int>(42);
    auto lambda2 = [p = std::move(ptr)]() {
        return *p;
    };
    std::cout << lambda2() << "\n";  // 42
    // ptr 现在为空
    
    // 捕获表达式结果
    auto lambda3 = [sum = x + 20]() {
        return sum;
    };
    std::cout << lambda3() << "\n";  // 30
}
```

---

### 3. 返回类型推导

函数返回类型可以使用 `auto`。

```cpp
#include <iostream>

// 简单返回类型推导
auto add(int a, int b) {
    return a + b;  // 返回类型推导为 int
}

// 多个 return 语句（类型必须一致）
auto get_value(bool flag) {
    if (flag) {
        return 42;
    } else {
        return 0;
    }
}

// 递归函数需要指定返回类型或使用尾置返回类型
auto factorial(int n) -> int {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// decltype(auto) 保留引用属性
decltype(auto) get_ref(int& x) {
    return x;  // 返回 int&
}

void return_type_deduction_example() {
    std::cout << add(3, 4) << "\n";
    std::cout << get_value(true) << "\n";
    std::cout << factorial(5) << "\n";
    
    int x = 10;
    get_ref(x) = 20;  // 可以修改 x
    std::cout << x << "\n";  // 20
}
```

---

### 4. 二进制字面量和数字分隔符

```cpp
#include <iostream>

void binary_literal_example() {
    // 二进制字面量
    int binary = 0b1010'1100;  // 172
    std::cout << "Binary: " << binary << "\n";
    
    // 数字分隔符（提高可读性）
    int million = 1'000'000;
    long long big = 1'234'567'890'123LL;
    double pi = 3.141'592'653'589;
    
    std::cout << "Million: " << million << "\n";
    std::cout << "Big: " << big << "\n";
    std::cout << "Pi: " << pi << "\n";
    
    // 十六进制也可以使用
    int hex = 0xDEAD'BEEF;
    std::cout << "Hex: " << std::hex << hex << std::dec << "\n";
}
```

---

### 5. std::make_unique

C++14 添加了 `make_unique`。

```cpp
#include <iostream>
#include <memory>

void make_unique_example() {
    // C++14 添加
    auto ptr1 = std::make_unique<int>(42);
    auto ptr2 = std::make_unique<std::string>("hello");
    
    // 数组版本
    auto arr = std::make_unique<int[]>(10);
    for (int i = 0; i < 10; ++i) {
        arr[i] = i;
    }
    
    std::cout << *ptr1 << "\n";
    std::cout << *ptr2 << "\n";
    std::cout << arr[5] << "\n";
}
```

---

## C++17 新特性

### 1. 结构化绑定

解包 tuple、pair、struct 等。

```cpp
#include <iostream>
#include <tuple>
#include <map>
#include <string>

struct Point {
    int x, y;
};

void structured_binding_example() {
    // 解包 tuple
    std::tuple<int, double, std::string> t(42, 3.14, "hello");
    auto [x, y, z] = t;
    std::cout << x << ", " << y << ", " << z << "\n";
    
    // 解包 pair
    std::pair<int, std::string> p{1, "one"};
    auto [id, name] = p;
    std::cout << id << ": " << name << "\n";
    
    // 解包 struct
    Point pt{10, 20};
    auto [px, py] = pt;
    std::cout << "Point: (" << px << ", " << py << ")\n";
    
    // 解包数组
    int arr[] = {1, 2, 3};
    auto [a, b, c] = arr;
    std::cout << a << ", " << b << ", " << c << "\n";
    
    // 用于 map 迭代（非常有用！）
    std::map<std::string, int> m{{"one", 1}, {"two", 2}, {"three", 3}};
    for (const auto& [key, value] : m) {
        std::cout << key << ": " << value << "\n";
    }
    
    // 修改值（使用非 const 引用）
    Point pt2{100, 200};
    auto& [x2, y2] = pt2;
    x2 = 300;
    std::cout << "Modified point: (" << pt2.x << ", " << pt2.y << ")\n";
}
```

---

### 2. if/switch 初始化语句

在 if 和 switch 中声明变量。

```cpp
#include <iostream>
#include <map>
#include <string>

void init_statement_example() {
    std::map<std::string, int> m{{"one", 1}, {"two", 2}};
    
    // if 带初始化（变量作用域仅限于 if 语句）
    if (auto it = m.find("one"); it != m.end()) {
        std::cout << "Found: " << it->second << "\n";
    } else {
        std::cout << "Not found\n";
    }
    // it 在这里不可见
    
    // switch 带初始化
    auto get_value = []() { return 42; };
    
    switch (auto val = get_value(); val) {
        case 42:
            std::cout << "The answer is " << val << "\n";
            break;
        default:
            std::cout << "Unknown value: " << val << "\n";
    }
    
    // 实际应用：检查并使用返回值
    if (auto [iter, success] = m.insert({"three", 3}); success) {
        std::cout << "Inserted: " << iter->first << "\n";
    }
}
```

---

### 3. std::optional

表示可能有值也可能没有值。

```cpp
#include <iostream>
#include <optional>
#include <string>

std::optional<int> divide(int a, int b) {
    if (b == 0) {
        return std::nullopt;  // 没有值
    }
    return a / b;  // 有值
}

std::optional<std::string> find_user(int id) {
    if (id == 1) {
        return "Alice";
    }
    return std::nullopt;
}

void optional_example() {
    // 基本用法
    auto result1 = divide(10, 2);
    if (result1) {  // 或 result1.has_value()
        std::cout << "Result: " << *result1 << "\n";  // 5
    }
    
    auto result2 = divide(10, 0);
    if (!result2) {
        std::cout << "Division by zero\n";
    }
    
    // value_or 提供默认值
    std::cout << divide(10, 2).value_or(-1) << "\n";  // 5
    std::cout << divide(10, 0).value_or(-1) << "\n";  // -1
    
    // value() 在没有值时抛出异常
    try {
        std::cout << divide(10, 0).value() << "\n";
    } catch (const std::bad_optional_access& e) {
        std::cout << "Exception: " << e.what() << "\n";
    }
    
    // 使用结构化绑定
    if (auto user = find_user(1); user) {
        std::cout << "User: " << *user << "\n";
    }
    
    // 创建 optional
    std::optional<int> opt1;              // 空
    std::optional<int> opt2 = 42;         // 有值
    std::optional<int> opt3 = std::nullopt;  // 空
    std::optional<int> opt4{std::in_place, 100};  // 原地构造
    
    // 赋值
    opt1 = 10;
    opt1 = std::nullopt;  // 清空
    opt1.emplace(20);     // 原地构造
    
    // 比较
    std::optional<int> a = 10;
    std::optional<int> b = 20;
    std::cout << "a < b: " << (a < b) << "\n";
}
```

---

### 4. std::variant

类型安全的 union。

```cpp
#include <iostream>
#include <variant>
#include <string>

void variant_example() {
    // 创建 variant（可以存储多种类型之一）
    std::variant<int, double, std::string> v;
    
    // 赋值不同类型
    v = 42;
    std::cout << "int: " << std::get<int>(v) << "\n";
    
    v = 3.14;
    std::cout << "double: " << std::get<double>(v) << "\n";
    
    v = "hello";
    std::cout << "string: " << std::get<std::string>(v) << "\n";
    
    // 检查当前类型
    std::cout << "Index: " << v.index() << "\n";  // 2 (string 是第 3 个类型)
    
    if (std::holds_alternative<std::string>(v)) {
        std::cout << "Currently holds string\n";
    }
    
    // 安全访问（如果类型不匹配会抛出异常）
    try {
        std::cout << std::get<int>(v) << "\n";
    } catch (const std::bad_variant_access& e) {
        std::cout << "Wrong type: " << e.what() << "\n";
    }
    
    // get_if 返回指针（失败返回 nullptr）
    if (auto p = std::get_if<std::string>(&v)) {
        std::cout << "String value: " << *p << "\n";
    }
    
    // 访问器模式（推荐）
    std::visit([](auto&& arg) {
        using T = std::decay_t<decltype(arg)>;
        if constexpr (std::is_same_v<T, int>) {
            std::cout << "Visiting int: " << arg << "\n";
        } else if constexpr (std::is_same_v<T, double>) {
            std::cout << "Visiting double: " << arg << "\n";
        } else if constexpr (std::is_same_v<T, std::string>) {
            std::cout << "Visiting string: " << arg << "\n";
        }
    }, v);
    
    // 重载访问器
    struct Visitor {
        void operator()(int i) const { 
            std::cout << "Int: " << i << "\n"; 
        }
        void operator()(double d) const { 
            std::cout << "Double: " << d << "\n"; 
        }
        void operator()(const std::string& s) const { 
            std::cout << "String: " << s << "\n"; 
        }
    };
    
    v = 100;
    std::visit(Visitor{}, v);
}
```

---

### 5. std::string_view

非拥有的字符串视图，避免拷贝。

```cpp
#include <iostream>
#include <string_view>
#include <string>

void print(std::string_view sv) {
    std::cout << sv << "\n";
}

std::string_view get_prefix(std::string_view sv, size_t n) {
    return sv.substr(0, n);
}

void string_view_example() {
    // 不拷贝原始字符串
    std::string str = "Hello, World!";
    print(str);           // 从 string
    print("Hello");       // 从字面量
    
    const char* cstr = "C string";
    print(cstr);          // 从 C 字符串
    
    // 子串操作（不拷贝）
    std::string_view sv = str;
    std::string_view sub = sv.substr(0, 5);  // "Hello"
    std::cout << sub << "\n";
    
    // 前缀/后缀
    std::cout << get_prefix("Hello, World!", 5) << "\n";
    
    // 比较
    std::string_view sv1 = "hello";
    std::string_view sv2 = "world";
    std::cout << "sv1 < sv2: " << (sv1 < sv2) << "\n";
    
    // 查找
    std::string_view text = "The quick brown fox";
    size_t pos = text.find("quick");
    if (pos != std::string_view::npos) {
        std::cout << "Found at: " << pos << "\n";
    }
    
    // 注意：string_view 不拥有数据
    std::string_view danger;
    {
        std::string temp = "temporary";
        danger = temp;  // 危险！
    }  // temp 被销毁
    // std::cout << danger << "\n";  // 未定义行为！
}
```

**注意事项：**
- `string_view` 不拥有数据
- 确保底层字符串在使用期间有效
- 不能修改字符串内容

---

### 6. std::filesystem

文件系统操作库。

```cpp
#include <iostream>
#include <filesystem>
#include <fstream>

namespace fs = std::filesystem;

void filesystem_example() {
    // 路径操作
    fs::path p = "/home/user/documents/file.txt";
    
    std::cout << "Filename: " << p.filename() << "\n";
    std::cout << "Extension: " << p.extension() << "\n";
    std::cout << "Parent path: " << p.parent_path() << "\n";
    std::cout << "Stem: " << p.stem() << "\n";
    
    // 路径拼接
    fs::path dir = "/home/user";
    fs::path file = dir / "documents" / "file.txt";
    std::cout << "Full path: " << file << "\n";
    
    // 文件存在性检查
    if (fs::exists(file)) {
        std::cout << "File exists\n";
        
        // 文件大小
        std::cout << "Size: " << fs::file_size(file) << " bytes\n";
        
        // 文件类型
        if (fs::is_regular_file(file)) {
            std::cout << "Regular file\n";
        }
        if (fs::is_directory(file)) {
            std::cout << "Directory\n";
        }
    }
    
    // 创建目录
    fs::path new_dir = "/tmp/test_dir";
    if (!fs::exists(new_dir)) {
        fs::create_directory(new_dir);
        std::cout << "Directory created\n";
    }
    
    // 创建多级目录
    fs::create_directories("/tmp/a/b/c");
    
    // 文件操作
    fs::path src = "/tmp/source.txt";
    fs::path dst = "/tmp/dest.txt";
    
    // 创建测试文件
    std::ofstream(src) << "test content";
    
    // 复制文件
    fs::copy_file(src, dst, fs::copy_options::overwrite_existing);
    
    // 重命名/移动
    fs::rename(dst, "/tmp/renamed.txt");
    
    // 删除文件
    fs::remove(src);
    
    // 删除目录及其内容
    fs::remove_all(new_dir);
    
    // 目录遍历
    fs::path dir_to_scan = "/tmp";
    if (fs::exists(dir_to_scan)) {
        std::cout << "\nContents of " << dir_to_scan << ":\n";
        for (const auto& entry : fs::directory_iterator(dir_to_scan)) {
            std::cout << entry.path() << "\n";
        }
    }
    
    // 递归遍历
    std::cout << "\nRecursive contents:\n";
    for (const auto& entry : fs::recursive_directory_iterator(dir_to_scan)) {
        std::cout << entry.path() << "\n";
    }
    
    // 获取当前路径
    std::cout << "Current path: " << fs::current_path() << "\n";
    
    // 获取临时目录
    std::cout << "Temp directory: " << fs::temp_directory_path() << "\n";
}
```

---

### 7. 折叠表达式

简化变参模板。

```cpp
#include <iostream>

// 一元右折叠
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);  // ((arg1 + arg2) + arg3) + ...
}

// 一元左折叠
template<typename... Args>
auto sum_left(Args... args) {
    return (... + args);  // ... + ((arg3 + arg2) + arg1)
}

// 二元右折叠
template<typename... Args>
auto sum_with_init(int init, Args... args) {
    return (init + ... + args);  // init + (arg1 + (arg2 + arg3))
}

// 打印所有参数
template<typename... Args>
void print(Args... args) {
    ((std::cout << args << " "), ...);  // 二元左折叠
    std::cout << "\n";
}

// 逻辑运算
template<typename... Args>
bool all(Args... args) {
    return (args && ...);  // 全部为 true
}

template<typename... Args>
bool any(Args... args) {
    return (args || ...);  // 任意一个为 true
}

void fold_expression_example() {
    std::cout << "Sum: " << sum(1, 2, 3, 4, 5) << "\n";  // 15
    std::cout << "Sum left: " << sum_left(1, 2, 3, 4, 5) << "\n";  // 15
    std::cout << "Sum with init: " << sum_with_init(100, 1, 2, 3) << "\n";  // 106
    
    print(1, 2.5, "hello", 'c');
    
    std::cout << "All true: " << all(true, true, true) << "\n";  // 1
    std::cout << "All true: " << all(true, false, true) << "\n";  // 0
    std::cout << "Any true: " << any(false, false, true) << "\n";  // 1
}
```

**折叠表达式类型：**
- `(args op ...)` - 一元右折叠
- `(... op args)` - 一元左折叠
- `(init op ... op args)` - 二元右折叠
- `(args op ... op init)` - 二元左折叠

---

## C++20 新特性

### 1. Concepts（概念）

约束模板参数。

```cpp
#include <iostream>
#include <concepts>
#include <vector>

// 定义概念
template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template<typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::convertible_to<T>;
};

// 使用概念约束模板
template<Numeric T>
T add(T a, T b) {
    return a + b;
}

// 使用 requires 子句
template<typename T>
requires std::integral<T>
T multiply(T a, T b) {
    return a * b;
}

// 尾置 requires
template<typename T>
T divide(T a, T b) requires std::floating_point<T> {
    return a / b;
}

// 复杂概念
template<typename T>
concept Container = requires(T t) {
    typename T::value_type;
    typename T::iterator;
    { t.begin() } -> std::same_as<typename T::iterator>;
    { t.end() } -> std::same_as<typename T::iterator>;
    { t.size() } -> std::convertible_to<std::size_t>;
};

template<Container C>
void print_container(const C& container) {
    for (const auto& item : container) {
        std::cout << item << " ";
    }
    std::cout << "\n";
}

void concepts_example() {
    std::cout << add(3, 4) << "\n";        // OK: int
    std::cout << add(3.5, 4.5) << "\n";    // OK: double
    // std::cout << add("hello", "world") << "\n";  // 错误：不满足 Numeric
    
    std::cout << multiply(5, 6) << "\n";   // OK: int
    // std::cout << multiply(5.5, 6.6) << "\n";  // 错误：不满足 integral
    
    std::cout << divide(10.0, 3.0) << "\n";  // OK: double
    
    std::vector<int> vec = {1, 2, 3, 4, 5};
    print_container(vec);  // OK: vector 满足 Container
}
```

---

### 2. Ranges

范围库，简化算法操作。

```cpp
#include <iostream>
#include <vector>
#include <ranges>
#include <algorithm>

void ranges_example() {
    std::vector<int> vec = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    // 过滤偶数
    auto even = vec | std::views::filter([](int x) { return x % 2 == 0; });
    std::cout << "Even numbers: ";
    for (int x : even) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    
    // 转换（平方）
    auto squared = vec | std::views::transform([](int x) { return x * x; });
    std::cout << "Squared: ";
    for (int x : squared) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    
    // 组合：过滤 + 转换
    auto result = vec 
        | std::views::filter([](int x) { return x % 2 == 0; })
        | std::views::transform([](int x) { return x * x; });
    
    std::cout << "Even squared: ";
    for (int x : result) {
        std::cout << x << " ";  // 4 16 36 64 100
    }
    std::cout << "\n";
    
    // 取前 N 个
    auto first_five = vec | std::views::take(5);
    std::cout << "First 5: ";
    for (int x : first_five) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    
    // 跳过前 N 个
    auto skip_five = vec | std::views::drop(5);
    std::cout << "Skip 5: ";
    for (int x : skip_five) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    
    // 反转
    auto reversed = vec | std::views::reverse;
    std::cout << "Reversed: ";
    for (int x : reversed) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    
    // 范围算法
    std::ranges::sort(vec, std::greater<>{});  // 降序排序
    std::cout << "Sorted desc: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << "\n";
}
```

---

### 3. 三路比较运算符（<=>）

自动生成所有比较运算符。

```cpp
#include <iostream>
#include <compare>
#include <string>

struct Point {
    int x, y;
    
    // 自动生成 ==, !=, <, <=, >, >=
    auto operator<=>(const Point&) const = default;
};

struct Person {
    std::string name;
    int age;
    
    // 自定义比较逻辑
    auto operator<=>(const Person& other) const {
        if (auto cmp = name <=> other.name; cmp != 0) {
            return cmp;
        }
        return age <=> other.age;
    }
    
    // 需要显式定义 ==
    bool operator==(const Person& other) const = default;
};

void three_way_comparison_example() {
    Point p1{1, 2};
    Point p2{3, 4};
    Point p3{1, 2};
    
    std::cout << "p1 == p3: " << (p1 == p3) << "\n";  // true
    std::cout << "p1 != p2: " << (p1 != p2) << "\n";  // true
    std::cout << "p1 < p2: " << (p1 < p2) << "\n";    // true
    std::cout << "p1 <= p3: " << (p1 <= p3) << "\n";  // true
    std::cout << "p2 > p1: " << (p2 > p1) << "\n";    // true
    std::cout << "p2 >= p1: " << (p2 >= p1) << "\n";  // true
    
    Person alice{"Alice", 30};
    Person bob{"Bob", 25};
    Person alice2{"Alice", 30};
    
    std::cout << "alice == alice2: " << (alice == alice2) << "\n";
    std::cout << "alice < bob: " << (alice < bob) << "\n";
}
```

---

### 4. std::span

非拥有的连续内存视图。

```cpp
#include <iostream>
#include <span>
#include <vector>
#include <array>

void print_span(std::span<int> s) {
    for (int x : s) {
        std::cout << x << " ";
    }
    std::cout << "\n";
}

void modify_span(std::span<int> s) {
    for (int& x : s) {
        x *= 2;
    }
}

void span_example() {
    // 从 C 数组
    int arr[] = {1, 2, 3, 4, 5};
    print_span(arr);
    
    // 从 vector
    std::vector<int> vec = {6, 7, 8, 9, 10};
    print_span(vec);
    
    // 从 array
    std::array<int, 5> std_arr = {11, 12, 13, 14, 15};
    print_span(std_arr);
    
    // 子范围
    std::span<int> full_span(arr);
    std::span<int> sub_span = full_span.subspan(1, 3);  // 从索引 1 开始，取 3 个
    std::cout << "Subspan: ";
    print_span(sub_span);
    
    // 修改原始数据
    modify_span(vec);
    std::cout << "Modified vector: ";
    print_span(vec);
    
    // 固定大小的 span
    std::span<int, 5> fixed_span(arr);
    std::cout << "Fixed size: " << fixed_span.size() << "\n";
}
```

---

### 5. std::format（格式化库）

类型安全的字符串格式化。

```cpp
#include <iostream>
#include <format>
#include <string>

void format_example() {
    // 基本格式化
    std::string s1 = std::format("Hello, {}!", "World");
    std::cout << s1 << "\n";
    
    // 位置参数
    std::string s2 = std::format("{0} + {1} = {2}", 1, 2, 3);
    std::cout << s2 << "\n";
    
    // 格式说明符
    std::string s3 = std::format("Pi = {:.2f}", 3.14159);  // 保留 2 位小数
    std::cout << s3 << "\n";
    
    // 宽度和对齐
    std::string s4 = std::format("|{:<10}|{:^10}|{:>10}|", "left", "center", "right");
    std::cout << s4 << "\n";
    
    // 十六进制
    std::string s5 = std::format("Hex: {:#x}", 255);
    std::cout << s5 << "\n";  // 0xff
    
    // 二进制
    std::string s6 = std::format("Binary: {:#b}", 42);
    std::cout << s6 << "\n";  // 0b101010
    
    // 填充字符
    std::string s7 = std::format("{:*^20}", "centered");
    std::cout << s7 << "\n";  // ******centered******
}
```

---

### 6. Coroutines（协程）

前面已经详细讲解过了，这里简单回顾：

```cpp
// 协程示例（需要自定义 promise_type）
#include <iostream>
#include <coroutine>

struct Generator {
    struct promise_type {
        int current_value;
        
        Generator get_return_object() {
            return Generator{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        
        std::suspend_always yield_value(int value) {
            current_value = value;
            return {};
        }
        
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };
    
    std::coroutine_handle<promise_type> handle;
    
    explicit Generator(std::coroutine_handle<promise_type> h) : handle(h) {}
    ~Generator() { if (handle) handle.destroy(); }
    
    int get_value() { return handle.promise().current_value; }
    bool next() {
        handle.resume();
        return !handle.done();
    }
};

Generator fibonacci(int count) {
    int a = 0, b = 1;
    for (int i = 0; i < count; ++i) {
        co_yield a;
        int next = a + b;
        a = b;
        b = next;
    }
}

void coroutine_example() {
    auto gen = fibonacci(10);
    while (gen.next()) {
        std::cout << gen.get_value() << " ";
    }
    std::cout << "\n";
}
```

---

## 总结

### C++11 核心改进
- 类型推导（auto, decltype）
- 移动语义和右值引用
- Lambda 表达式
- 智能指针
- 多线程支持
- 变参模板

### C++14 增强
- 泛型 Lambda
- 返回类型推导
- 二进制字面量

### C++17 实用特性
- 结构化绑定
- std::optional, std::variant
- std::string_view
- std::filesystem
- 折叠表达式

### C++20 现代化
- Concepts
- Ranges
- Coroutines
- 三路比较
- std::span
- std::format

---

## 编译和使用

```bash
# C++11
g++ -std=c++11 program.cpp -o program

# C++14
g++ -std=c++14 program.cpp -o program

# C++17
g++ -std=c++17 program.cpp -o program

# C++20
g++ -std=c++20 program.cpp -o program

# 启用所有警告
g++ -std=c++20 -Wall -Wextra program.cpp -o program

# 优化
g++ -std=c++20 -O3 program.cpp -o program
```

---

**建议学习路径：**
1. 先掌握 C++11 核心特性（auto, lambda, 智能指针）
2. 学习 C++14 的改进（泛型 lambda, make_unique）
3. 掌握 C++17 实用特性（结构化绑定, optional, filesystem）
4. 最后学习 C++20 高级特性（concepts, ranges, coroutines）
