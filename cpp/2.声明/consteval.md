# 立即函数

使用 `consteval` 声明的函数，函数调用必须生成编译期常量。

> 弱化了 `constepxr`，使用 `consteval` 替代。

# if consteval

C++23 引入，基本语法为 `if consteval { }`，C++20 的等价代码为 `if(std::is_const_evaluated()) { }`。

```cpp
consteval auto const_evaluate(int n) -> int { return n; }

auto nonconst_evaluate(int n) -> int { return n * n; }

constexpr auto func(int n) -> int {
    if consteval {
        return const_evaluate(n);
    } else {
        return nonconst_evaluate(n);
    }
}

constexpr auto i = func(10);

auto main() -> int {
    std::cout << i << " " << func(10);
    return 0;
}
```
