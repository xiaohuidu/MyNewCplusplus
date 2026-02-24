一、Concepts 是为了解决什么问题？

在 C++20 之前，模板有三个致命痛点：

1️⃣ 错误信息灾难
std::sort(x.begin(), x.end());

如果 x 不满足要求，错误信息可能几百行。

2️⃣ 约束是“隐式”的
template<typename T>
void f(T t) {
    t.size();   // 隐式要求：T 必须有 size()
}

👉 只有实例化失败时才知道要求。

3️⃣ SFINAE 写法晦涩
template<typename T,
         typename = std::enable_if_t<
             std::is_integral_v<T>
         >>
void f(T);

难读、难写、难维护。

👉 Concepts 的目标

把“模板参数的要求”变成语言的一部分
让错误更早、更清晰、更可读

二、Concepts 是什么？
定义（直观）

Concept 是一个“对模板参数的编译期谓词（constraint）”

Concept 本质上是：

template<typename T>
concept XXX = /* 一个编译期 bool 表达式 */;
一个最简单的 Concept
template<typename T>
concept Integral = std::is_integral_v<T>;

使用：

template<Integral T>
T add(T a, T b) {
    return a + b;
}
三、Concept 的基本语法
1️⃣ 定义 Concept
template<typename T>
concept Name = expression;

expression 必须是：

编译期

可转换为 bool

2️⃣ 使用 Concept（4 种方式）
✅ 方式 1：模板参数列表（推荐）
template<Integral T>
void f(T);
✅ 方式 2：requires 子句
template<typename T>
requires Integral<T>
void f(T);
✅ 方式 3：混合多个 Concept
template<typename T>
requires Integral<T> && std::is_signed_v<T>
void f(T);
✅ 方式 4：auto + Concept（最现代）
void f(Integral auto x);
四、requires 表达式（Concept 的灵魂）
1️⃣ 最简单的 requires
template<typename T>
concept HasSize = requires(T t) {
    t.size();
};

意思：

只要 t.size() 是合法表达式即可

2️⃣ requires 中可以写什么？
✔ 表达式要求
{ t.size() };
✔ 返回类型要求
{ t.size() } -> std::same_as<std::size_t>;
✔ noexcept 要求
{ t.clear() } noexcept;
✔ 类型要求
typename T::value_type;
3️⃣ requires 的“语义特性”

不执行

不实例化函数体

只检查“能不能写出来”

五、Concepts vs SFINAE（核心对比）
对比项	SFINAE	Concepts
可读性	极差	极好
错误信息	爆炸	精准
语义表达	隐式	显式
组合能力	困难	自然
语言支持	技巧	语言级
六、Concepts 与 decltype / declval 的关系
1️⃣ Concepts 取代的是 SFINAE
旧：
decltype + declval + enable_if + SFINAE

新：
concept + requires
2️⃣ Concepts 不负责类型推导
using size_type = decltype(std::declval<T>().size());

👉 这仍然需要 decltype / declval

七、Concepts 的“短路”和重载决议（非常重要）
1️⃣ Concept 是短路的
template<typename T>
concept C =
    std::is_integral_v<T> &&
    requires(T t) { t.foo(); };

若第一个条件失败，后面不会检查。

2️⃣ Concept 参与重载排序
template<typename T>
void f(T);          // #1

template<Integral T>
void f(T);          // #2

调用：

f(10);   // 选 #2

👉 约束更严格的优先

八、标准库 Concepts（必须知道）
#include <concepts>
Concept	含义
std::integral	整数类型
std::floating_point	浮点
std::same_as<T>	类型相同
std::derived_from	继承
std::convertible_to<T>	可转换
std::invocable	可调用
std::regular	类似普通对象
std::ranges::range	范围
九、Concepts 在 STL 中的真实样子
std::sort（概念化）
template<
    std::random_access_iterator I,
    std::sentinel_for<I> S,
    class Comp = std::ranges::less
>
void sort(I first, S last, Comp comp = {});

📌 读起来就是接口文档。

十、Concepts 的设计哲学（重要）
1️⃣ “语义”比“语法”重要
concept Addable = requires(T a, T b) {
    a + b;
};

不是“能编译就行”，而是“表达意图”。

2️⃣ Concept 应该小而可组合
concept Addable = ...
concept Comparable = ...

concept Number = Addable && Comparable;
十一、最佳实践（工程级）
✅ 优先写 Concept，而不是 requires 子句
// 好
template<typename T>
concept Indexable = requires(T t) {
    t[0];
};
✅ Concept 名用形容词 / 名词

Sortable

Range

Callable

❌ 不在 Concept 中做复杂计算

Concept 是“接口”，不是“算法”。

十二、一句话终极总结

Concepts 把模板的“隐式约定”变成了“显式接口”，
把 SFINAE 从技巧升级成了语言。
