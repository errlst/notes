[tuple<Types...>]()提供在一个单元中存放任意数量固定大小的异构元素的方法。

## 成员访问

如果要访问[tuple]()成员，可以通过[std::get<Idx\>()]()提取第_Idx_个元素，也可以通过[结构化绑定]()一次提取所有元素。

## 辅助函数

[std::make_tuple(args...)]()，创建[tuple]()对象。

[std::tie(args...)]()，创建[tuple]()对象，且对象保存的均为传入参数的引用。通过这个函数，可以快速将[tuple]()对象的元素提取到已存在的变量中，且可传入[std::ignore]()对象[^1]表示忽略某个元素。[示例](#示例1)

[std::forward_as_tuple(args...)]()，以[tuple]()的方式进行转发。

[std::tuple_cat(tuples...)]()，拼接任意数量的[tuple]()。

# 示例

## 示例1

```cpp
auto main() -> int {
    int a{}, b{};
    std::tie(a, b, std::ignore) = std::make_tuple(1, 2, 3);
    std::cout << a << " " << b; // 1 2

    return 0;
}
```

[^1]:[ignore]()对象的类型，[operator=]()可接受任意类型参数，且不会做任何处理