[integer_sequence<T, T... Ints>]()，表示编译期整数序列，该类自身通常无意义，而是将_Ints_用于包展开。

使用[make_integer_sequence<T, N>]()别名，表示[integer_sequence<T, 0...N-1>]()。

# 示例

```cpp
template <size_t... Ints>
constexpr auto call(std::index_sequence<Ints...>) -> void {
    ((std::cout << Ints << " "), ...);
}

auto main() -> int {
    call(std::make_index_sequence<2>{});  // 0 1
    return 0;
}
```

