<link href="../../../style.css", rel="stylesheet">

# 自定义格式化

为了实现自定义类型的格式化，需要定义 `formatter` 对该类型的特化。每个 `formatter` 必须声明 `parse()` 和 `format()` 两个函数，可以定义为成员函数，也可以定义为静态函数。

<div class="code_block">
<div>
示例：

```cpp
struct MyInt {
  int val;
};

/* 限制输出在 0~100 范围内*/
template <>
struct std::formatter<MyInt> : public std::formatter<int> {
  auto format(const MyInt &v, std::format_context &fmt_ctx) const -> decltype(auto) {
    return std::formatter<int>::format(std::clamp(v.val, 0, 100), fmt_ctx);
  }
};

auto main() -> int {
  std::cout << std::format("{} {} {}\n", MyInt{-1}, MyInt{10}, MyInt{200});
  return 0;
}
```

</div>

<div>
输出：

```shell
0 10 100
```

</div>
</div>

### parse

`parse()` 函数执行读取类型格式规范的工作。其应该将格式规范中的所有格式信息存储在 `formatter` 对象本身上。其原型一般为：

```cpp
constexpr auto parse(format_parse_context& fpc) -> format_parse_context::iterator;

static constexpr auto parse(format_parse_context& fpc) -> format_parse_context::iterator;
```

格式化字符串中的替换字段一般格式为：`{[arg-id] [:fmt-spec]}`，因此格式规范是 `:` 之后、`}` 之前的所有内容。

进入函数时，`pc.begin()` 指向格式规范开头，`pc.end()` 指向 `}`，且标准指定空格式规范可以通过 `pc.begin() == pc.end()` 或 `*pc.begin() == '}'` 方式指示。

如果格式规范处理存在错误，因为 `parse()` 需要在编译期完成，因此报告错误将会很复杂。通常可以直接抛出 `format_error` 异常，直接让编译器报错。

如果格式规范的处理没有错误，则函数应该返回指向 `}` 的迭代器，即 `fpc.end()`。

### format

`format()` 函数输出实际替换字段的数据，且考虑 `parse()` 处理的格式规范。其原型一般为：

```cpp
// 注意需要定义为 const 成员函数
auto format(const T& t, format_context& fc) const -> format_context::iterator;

static auto format(const T& t, format_context& fc) -> format_context::iterator;
```

写入数据时，通常借助 `format_to()` 函数，使用标准格式化，将格式化后的内容写入写入 `fc.out()` 迭代器，并返回更新的迭代器作为结果。其返回值可以直接作为 `format()` 的返回值。

# 运行时格式化串

`std::format()` 格式化时，必须使用字面量常量作为格式化串。如果需要使用运行时字符串作为格式化串，一种方式是使用类型擦除的格式化函数，如 `std::vformat`。

C++26 引入 `std::runtime_format`，生成运行时的格式化串，更方便使用。

<div class="code_block">

<div>
before:

```cpp
auto main() -> int {
  auto fmt = "{}";
  auto str = std::vformat(fmt, std::make_format_args("hello"));
  return 0;
}
```

</div>

<div>
after:

```cpp
auto main() -> int {
  auto fmt = "{}";
  auto str = std::format(std::runtime_format(fmt), "hello");
  std::println(std::runtime_format(fmt), "hello");
  return 0;
}
```

</div>
</div>

# print

print 函数簇是对 format 库的更简易使用。

`print(fmt, args...)` 将格式化后的字符串输出到 _stdout_。

`print(stream, fmt, args...)` 将格式化后的字符串输出到文件流。
