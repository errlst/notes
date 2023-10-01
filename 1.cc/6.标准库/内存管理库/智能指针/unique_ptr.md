[unique_ptr<T, Deleter>]()是独占式智能指针。

[.reset(ptr)]()替换当前管理的资源，如果当前管理资源不为空，将调用删除器。

[.release()]()释放当前管理的资源，返回资源指针[^1]。

[.get()]()访问当前管理的资源，也可以通过[operator->]()或[operator*]()访问资源。

管理数组资源时，可以通过[operator[]]()访问数组元素。

## 构造

可以使用[std::make_unique()]()函数簇构造[unique_ptr]()对象，这样可以避免显示的[new]()表达式，但是无法设置删除器。

* [make_unique<T\>(args...)]()，构造非数组类型。
* [make_unique<T\>(size_t)]()，构造未知边界数组类型。

[make_unique()]()不提供构造已知边界数组类型。[示例](#示例)

# 示例

```cpp
auto main() -> int {
    auto p1 = std::make_unique<int>();
    auto p2 = std::make_unique<int[]>(3);
    auto p3 = std::make_unique<int[3]>();   // error

    return 0;
}
```

[^1]:需要调用方接受资源，或释放资源