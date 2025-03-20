`expected<T, E>` 提供存储二值之一的方法，其要么保有期待类型 _T_，要么保有非期待类型 _E_。

存储的值直接分配在 `expected` 对象的存储空间中，其结构类似：

```cpp
template<typename T, typename E>
class expected {
    union {
        T val_;
        E err_;
    };
    bool has_val_;
};
```

#### unexpected

在初始化或赋值 `expected` 对象时，如果需要将其设置为非期待类型，需要使用 `unexpected<E>` 作为中间对象。

或者使用 `std::unexpect` 作为第一个参数传递。

```cpp
auto main() -> int {
    std::cout << std::expected<int, int>{0}.has_value() << "\n";                       // true
    std::cout << std::expected<int, int>{std::unexpected<int>{0}}.has_value() << "\n"; // false
    std::cout << std::expected<int, int>{std::unexpect}.has_value() << "\n";		   // false

    return 0;
}
```

#### 访问

`.value()`，如果包含预期值，返回预期值；否则，抛出 `bad_expected_access` 异常，且其保有非预期值的副本。

`.error()`，如果包含预期值，行为未定义；否则，返回非预期值。

`.value_or(def_val)`，如果包含预期值，返回预期值；否则，返回 _def_val_。

#### monadic

`.and_then(f)`，如果包含期待值，返回 `f(value())`；否则，返回包含非预期值的副本的 `expected` 对象。且对 _f_ 的返回值类型 _R_，要么 _R_ 是 `expected` 的特化，要么 `same_as<R::error_type, E>`，否则非良构。

`.transform(f)`，`.and_then` 的简化版本，会将 _f_ 的返回类型封装为 `expected`。

```cpp
auto main() -> int {
    std::cout << std::expected<int, double>{10}.and_then([](int v) {
                                                   return std::expected<int, double>{v * v};
                                               }).value();
    std::cout << std::expected<int, double>{10}.transform([](int v) {
                                                   return v * v;
                                               }).value();
    return 0;
}
```

`.or_else(f)`，如果包含期待值，返回包含期待值的副本的 `expected` 对象，且对 _f_ 的返回值类型 _R_，要么 _R_ 是 `expected` 的特化，要么 `same_as<R::value_type, T>`，否则非良构。

`.transform_error(f)`，`.or_else` 的简化版本。



