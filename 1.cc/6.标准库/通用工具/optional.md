[optional<T\>]()管理可选的容纳值，其可转换为[bool]()类型，当其存在容纳值时为[true]()。

[optional]()管理的资源直接存放在该对象内部，而非通过动态内存分配。

使用[.value()]()获取其管理的资源，也可以直接通过[operator*]()或[operator->]()得到其管理资源的指针[^1]。

使用[.value_or(v)]()获取其管理资源，如果没有管理的资源，返回_v_。

使用[.reset()]()销毁其管理的资源[^2]，如果没有管理的资源，不会进行任何处理。[示例](#示例1)

使用[.emplace(args...)]()原地构造管理资源，如果当前已存在资源，将销毁当前资源。

# 示例

## 示例1

```cpp
auto main() -> int {
    struct T {
        ~T() { std::cout << "destruct\n"; }
    };
    auto op = std::optional{T{}};   // destruct
    std::cout << "-------\n";
    op.reset(); // destruct
    op.reset(); // 无输出

    return 0;
}
```





[^1]:当没有管理资源时，未定义行为
[^2]:只是调用析构函数，并设为无效