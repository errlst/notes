`ranges::to<C>(r, args...)` 从 r 的元素构造一个 C 类型的对象。

```cpp
auto main() -> int {
  auto numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9};

  auto even_numbers = numbers |
                      std::views::filter([](auto i) { return i % 2 == 0; }) |
                      std::ranges::to<std::vector>();
  std::println("{}", even_numbers);
  return 0;
}
```

将 `ranges::to` 用在管道语法时，括号是必须的。

# 优化

默认情况，将元素插入容器通常是通过复制完成的，因为间接调用时会产生左值引用。

```cpp
struct T {
  T() = default;
  T(const T &) { std::println("copy"); }
  T(T &&) { std::println("move"); }
};

auto main() -> int {
  T ts[2];
  auto ts_vec = ts | std::ranges::to<std::vector>();  // copy
  return 0;
}
```

可以通过 `views::as_rvalue` 来适配 ranges，使得调用时始终产生右值引用。

```cpp
struct T {
  T() = default;
  T(const T &) { std::println("copy"); }
  T(T &&) { std::println("move"); }
};

auto main() -> int {
  T ts[2];
  auto ts_vec = ts | std::views::as_rvalue | std::ranges::to<std::vector>(); // move
  return 0;
}
```
