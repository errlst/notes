`optional<T>` 管理可选的容纳值，其可转换为 `bool` 类型，当其包含有效值时为 `true`。`optional` 管理的资源直接存放在该对象内部，而非通过动态内存分配。

# 单子操作

C++23 为 optional 增加了一些单子操作函数：

- `and_then(f)`，如果有值，使用值调用 f，并返回其返回值；否则，返回 `nullopt`。

  > f 的返回类型必须是 `optional` 的特化。

- `transform(f)`，同上，但 f 的返回类型无特殊限制（非 `inplace_t` 或 `nullopt_t` 即可），且使用 `optional` 包裹返回的值。

- `or_else(f)`，如果有值，返回 `*this`；否则，返回 f 的调用结果。

```cpp
auto input_line() -> std::optional<std::string> {
  auto line = std::string{};
  std::print("input: ");
  std::getline(std::cin, line);
  if (line.empty()) {
    return std::nullopt;
  }
  return line;
}

auto main() -> int {
  auto res = input_line()
                 .transform([](auto line) -> std::string {
                   std::transform(line.begin(), line.end(), line.begin(), ::toupper);
                   return line;
                 })
                 .and_then([](auto line) -> std::optional<std::string> {
                   return "upper result: " + line;
                 })
                 .or_else([]() -> std::optional<std::string> {
                   return "no input";
                 });
  std::println("{}", *res);
  return 0;
}
```
