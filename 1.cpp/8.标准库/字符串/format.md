#### 自定义格式化
为了实现自定义类型的格式化，需要定义 `formatter` 对该类型的特化。每个 `formatter` 必须声明 `parse()` 和 `format()` 两个函数，可以定义为成员函数，也可以定义为静态函数。
```cpp
struct MyInt {
    int val;
};

// 格式化限制范围在 0~100
template<>
struct std::formatter<MyInt> : public std::formatter<int> {
    auto format(const MyInt &v, std::format_context &fmt_ctx) const {
        return std::formatter<int>::format(std::min(std::max(0, v.val), 100), fmt_ctx);
    }
};

int main() {
    std::cout << std::format("{} {} {}\n", MyInt{-1}, MyInt{10}, MyInt{200});
    return 0;
}
```

###### parse
`parse()` 函数执行读取类型格式规范的工作。其应该将格式规范中的所有格式信息存储在 `formatter` 对象本身上。其原型一般为：
```cpp
constexpr auto parse(format_parse_context& fpc) -> format_parse_context::iterator;

static constexpr auto parse(format_parse_context& fpc) -> format_parse_context::iterator;
```

格式化字符串中的替换字段一般格式为：`{[arg-id] [:fmt-spec]}`，因此格式规范是 `:` 之后、`}` 之前的所有内容。

进入函数时，`pc.begin()` 指向格式规范开头，`pc.end()` 指向 `}`，且标准指定空格式规范可以通过 `pc.begin() == pc.end()` 或 `*pc.begin() == '}'` 方式指示。

如果格式规范处理存在错误，因为 `parse()` 需要在编译期完成，因此报告错误将会很复杂。通常可以直接抛出 `format_error` 异常，直接让编译器报错。

如果格式规范的处理没有错误，则函数应该返回指向 `}` 的迭代器，即 `fpc.end()`。

###### format
`format()` 函数输出实际替换字段的数据，且考虑 `parse()` 处理的格式规范。其原型一般为：
```cpp
// 注意需要定义为 const 成员函数
auto format(const T& t, format_context& fc) const -> format_context::iterator;

static auto format(const T& t, format_context& fc) -> format_context::iterator;
```

写入数据时，通常借助 `format_to()` 函数，使用标准格式化，将格式化后的内容写入写入 `fc.out()` 迭代器，并返回更新的迭代器作为结果。其返回值可以直接作为 `format()` 的返回值。

###### demo
```cpp
struct Point {
    int x;
    int y;
};

template<>
struct std::formatter<Point> {
    static constexpr auto parse(std::format_parse_context &fpc) {
        if (fpc.begin() == fpc.end()) {
            throw std::format_error("only blank spec allowed");
        }
        return fpc.end();
    }

    static auto format(const Point &p, std::format_context &fc) {
        return std::format_to(fc.out(), "({}, {})", p.x, p.y);
    }
};

int main() {
    auto p1 = Point(1, 2);
    std::cout << std::format("{}", p1);
    return 0;
}
```