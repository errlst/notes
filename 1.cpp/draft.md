如果数组类型的元素类型是常量，那么这个数组类型就是常量。

```cpp
auto main() -> int {
    std::cout << std::is_const_v<int[]> << "\n";       // 0
    std::cout << std::is_const_v<const int[]> << "\n"; // 1
    return 0;
}
```

---

###### 属性

只有当需要引入属性说明符时，才能出现连续的两个 _[_ 符号。

```cpp
struct T {
    template <typename C>
    auto operator[](C c) {}
};

auto main() -> int {
    T {}[[]{}];  // error
    return 0;
}
```

---

在表达式语句和声明语句之间存在歧义时（使用函数样式的显示转换作为最左侧表达式），将其作为声明语句处理。

```cpp
int i;
auto main() -> int {
    do {
        int(i); // 声明局部变量 i
        i = 20;
        std::cout << ::i; // 0
    } while (0);
    do {
        int{i}; // 定义临时变量，使用 i 初始化
        i = 20;
        std::cout << ::i; // 20
    } while (0);
    return 0;
}
```

---

###### extern

```cpp
extern "C" int i; // 只是声明

extern "C" {
    int j; // 定义
}
```

---

###### 类型别名

解析声明说明符序列时，如果遇到类型名称，只有当类型名称前面不存在除了 cv 限定符之外的类型说明符时，其才解析为类型名称，否则被解析为标识符。

```cpp
// 接受 unsigned 类型的参数，i64 是参数名
auto f(unsigned i64) { std::cout << "1\n"; }

auto f(unsigned long long i) { std::cout << "2\n"; }

auto main() -> int {
    f(0ull);  // 2
    return 0;
}
```











