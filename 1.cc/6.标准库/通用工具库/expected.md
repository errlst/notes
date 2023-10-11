[expected<T, E>]()提供存储二值之一的方法，其要么保有期待类型_T_，要么保有非期待类型_E_。其中，_T_不能是[引用类型]()或[函数类型]()[^1]。其维护的值直接存放在[expected]()的存储空间中，其基本结构如下：

```cpp
template<typename T, typename E>
class expected {
    union {
        T m_val;
        E m_err;
    };
    bool m_has_val;
};
```

## 访问

使用[operator bool]()或[.has_value()]()判断当前是否存储期待值。

使用[operator->]()或[operator*]()获取期待值的指针或引用。如果此时没有存储期待值，行为未定义。

使用[.value()]()获取期待值，使用[.error]()返回非期待值。

使用[.emplace(args...)]()原位构造期待值。

[^1]:函数对象不是函数类型