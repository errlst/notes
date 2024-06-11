`expected<T, E>` 提供存储二值之一的方法，其要么保有期待类型 _T_，要么保有非期待类型 _E_。

存储的值直接分配在 `expected` 对象的存储空间中，其结构类似：

```cpp
template<typename T, typename E>
class expected {
    union {
        T m_val;
        E m_err;
    };
    bool m_has_val;
};
```

#### unexpected

在初始化或赋值 `expected` 对象时，如果需要将其设置为非期待类型，需要使用 `unexpected<E>` 作为中间对象。

```cpp
auto main() -> int {
    std::cout << std::expected<int, int>{0}.has_value() << "\n";                       // true
    std::cout << std::expected<int, int>{std::unexpected<int>{0}}.has_value() << "\n"; // false

    return 0;
}
```







## 访问

使用[operator bool]()或[.has_value()]()判断当前是否存储期待值。

使用[operator->]()或[operator*]()获取期待值的指针或引用。如果此时没有存储期待值，行为未定义。

使用[.value()]()获取期待值，使用[.error]()返回非期待值。

使用[.emplace(args...)]()原位构造期待值。



