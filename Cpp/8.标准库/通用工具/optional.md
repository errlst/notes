`optional<T>` 管理可选的容纳值，其可转换为 `bool` 类型，当其包含有效值时为 `true`。`optional` 管理的资源直接存放在该对象内部，而非通过动态内存分配。

#### 单子操作

`and_then(f)`，如果当前对象保有值，将值作为实参调用 _f_，并返回调用结果；否则，返回空的 `optional`。_f_ 的返回类型必须是 `optional` 的特化。

`transform(f)`，同上，但 _f_ 的返回类型无特殊限制（非 `inplace_t` 或 `nullopt_t` 即可），且使用 `optional` 包裹返回的值。

`or_else(f)`，如果当前对象保有值，返回 `*this`；否则，调用 _f_ 并返回其结果。

```cpp
auto main() -> int {
    auto op = std::optional{10};
    std::println("{}", op.and_then([](int i) { return std::optional{i * i}; }).value());
    std::println("{}", op.transform([](int i) { return i * i; }).value());
    op.reset();
    std::println("{}", op.or_else([] { return std::optional{100}; }).value());
    return 0;
}
```





使用[.value()]()获取其管理的资源，也可以直接通过[operator*]()或[operator->]()得到其管理资源的指针。

使用[.value_or(v)]()获取其管理资源，如果没有管理的资源，返回_v_。

使用[.reset()]()销毁其管理的资源[^2]，如果没有管理的资源，不会进行任何处理。[示例](#示例1)

使用[.emplace(args...)]()原地构造管理资源，如果当前已存在资源，将销毁当前资源。
