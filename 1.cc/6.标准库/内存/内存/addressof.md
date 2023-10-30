[addressof(obj)]()获取对象的真实地址，即使存在[&]()重载。

# 示例

```cpp
auto main() -> int {
    struct {
        auto operator&() { return nullptr; }
    } t;
    std::cout << (&t == nullptr) << "\n";                   // true
    std::cout << (std::addressof(t) == nullptr) << "\n";    // false

    return 0;
}
```

