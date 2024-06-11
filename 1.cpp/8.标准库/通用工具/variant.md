[variant<Types...>]()表示类型安全的联合体，其在任意时刻都会保留其一个变体对象，其基本实现如下：

```cpp
template<typename ... Types>
union variant_union;

template<typename T, typename ... Types>
union variant_union<T, Types...> {
    T val;
    variant_union<Types...> rest;
};

template<typename T>
union variant_union<T> {
    T val;
};

template<typename ... Types>
class variant {
    variant_union<Types...> data;
    size_t idx;
};
```

[variant]()的行为和联合体一致，所有变体对象直接存放在[variant]()内部，而非通过动态分配的方式存储。

可以存在多个相同类型的变体对象，但不能具有[引用]()、[数组]()或[void]()类型。

## 访问变体

### get

`std::get<>(v)`访问_v_中的指定变体，如果_v_当前有效的变体对象和需要访问的变体对象不匹配，将抛出异常：

* `get<Idx>(v)`，通过指定下标访问，下标必须合法。
* `get<Type>(v)`，通过指定类型访问，_Type_必须是_v_中存在的唯一的元素类型。

`std::get_if<>(v)`获取_v_中指定变体的指针，如果_v_当前有效的变体对象和需要访问的变体对象不匹配，返回空指针，而不是抛出异常。

### visit

`std::visit(visitor, variants...)`，将_variants_中的当前有效的变体对象，按序传递给可调用对象_visitor_。[示例](#示例1)

# 示例

## 示例1

```cpp
template<typename ... Visitors>
struct visitor_combine : Visitors ... { using Visitors::operator() ...; };

auto main() -> int {
    std::variant<int, float, double> v1{0}, v2{0.f}, v3{0.0};

    // 方式一
    auto visitor1 = [](auto &&arg) {
        using T = std::decay_t<decltype(arg)>;
        if constexpr (std::is_same_v<T, int>) {
            std::cout << "int type\n";
        } else if constexpr (std::is_same_v<T, double>) {
            std::cout << "double type\n";
        } else {
            std::cout << "unknown type\n";
        }
    };
    std::visit(visitor1, v1);   // int
    std::visit(visitor1, v2);   // unknown
    std::visit(visitor1, v3);   // double

    // 方式二
    auto visitor2 = visitor_combine{
        [](int) { std::cout << "int type\n"; },
        [](double) { std::cout << "double type\n"; },
        [](auto &&) { std::cout << "unknown type\n"; }
    };
    std::visit(visitor2, v1);   // int
    std::visit(visitor2, v2);   // unknown
    std::visit(visitor2, v3);   // double

    return 0;
}
```

