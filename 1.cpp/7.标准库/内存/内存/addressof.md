[toc]

# addressof

`std::addressof(o)`借助编译器内置函数，获取对象的真实地址，忽略`operator&`重载。

## 实现

```cpp
template <typename T>
constexpr auto addressof(T& t) noexcept -> T* {
    return __buildin_addressof(t);
}
```



