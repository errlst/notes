[derived_from<D, B>]()，忽略cv限定后，_B_是类类型，且其和_D_类型相等，或是_D_的公开基类时，满足概念。其基本实现如下：[示例](#示例1)

```cpp
template<typename D, typename B>
concept derived_from = std::is_base_of_v<B, D> &&
                       std::is_convertible_v<const volatile D *, const volatile B *>;
```

# 示例

## 示例1

```cpp
struct PublicBase {};

struct PrivateBase {};

struct Derive : public PublicBase, private PrivateBase {};

auto main() -> int {
    std::cout << std::derived_from<Derive, Derive> << "\n";         // 1
    std::cout << std::derived_from<Derive, PublicBase> << "\n";     // 1
    std::cout << std::derived_from<Derive, PrivateBase> << "\n";    // 0

    return 0;
}
```

