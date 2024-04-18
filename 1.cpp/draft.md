
---

某些情况下，需要借助偏特化进行的模板类型萃取时，还可以使用模板函数的返回值实现相同的效果，且可通过参数列表避免偏特化。

```cpp
// 萃取第一个类型
template <typename T, typename... Types>
constexpr auto first_type() -> T;

template <typename... Types>
using first_type_t = decltype(first_type<Types...>());

// 或者
template<typename T, typename ... Types>
struct first_type{
    using type = T;  
};

template<typename ... Types>
using first_type_t = first_type<Types...>::type;
```

---

`enum` 不能预先声明，而 `enum class` 可以。因此如果需要特化模板的枚举成员，必须使用 `enum class`。（也许是这个原因

```cpp
// OK
template<typename T>
struct U{
    enum class E{};
};

template<>
enum class U<int>::E{};

// Error
template<typename T>
struct U{
    enum E{};
};

template<>
enum U<int>::E{};
```