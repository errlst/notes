#### auto 和 模板
使用 `auto` 进行类型推断时，其推断行为和模板参数的推断行为几乎一致。
```cpp
auto main() -> int {
    int x = 0;
    auto&& a1 = x;  // x为左值，类型为int，因此a1推断为 int&
    std::cout << std::is_lvalue_reference_v<decltype(a1)> << "\n";
    auto&& a2 = 0;  // 0为右值，类型为int，因此a2推断为 int&&
    std::cout << std::is_rvalue_reference_v<decltype(a2)> << "\n";
    return 0;
}
```
但存在一个例外情况，即当使用 `={}` 进行初始化时，如果没有显示指定参数类型，会推断为 `initializer_list`，且如果参数类型不一致，将编译失败。
```cpp
auto main() -> int {
    auto a1(0);
    std::cout << (typeid(a1) == typeid(int)) << "\n";
    auto a2 = 0;
    std::cout << (typeid(a2) == typeid(int)) << "\n";
    auto a3{ 0 };
    std::cout << (typeid(a3) == typeid(int)) << "\n";
    auto a4 = { 0 };
    std::cout << (typeid(a4) == typeid(std::initializer_list<int>)) << "\n";
    return 0;
}
```

而在模板中，如果传递 `initializer_list` 对象，但模板参数没有显示指明为 `initializer_list`，那么将无法推断类型。
```cpp
template <typename T>
auto call_1(T&&) -> void {}

template <typename T>
auto call_2(std::initializer_list<T>) -> void {}

auto main() -> int {
//  call_1({ 1, 2, 3 });                        // error
//  call_1(std::initializer_list{ 1, 2, 3 });   // error
    call_1(std::initializer_list<int>{1, 2, 3});
    call_2({ 1, 2, 3 });
    return 0;
}
```
总结，`auto` 和模板推导的唯一区别：`auto` 会假定使用 `{}` 包裹的初始化器为 `initializer_list` 对象。