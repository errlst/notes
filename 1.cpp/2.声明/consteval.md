使用[consteval]()指定[立即函数]()，函数调用必须生成编译期常量。[consteval]()声明隐式蕴含[inline]()。

[consteval]()不能声明『析构函数』、『分配函数』和『释放函数』。

#### if consteval

`if consteval { do_a } else { ao_b }` 如果当前处于常量求值，执行 _do_a_，否则，执行 _do_b_。

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

