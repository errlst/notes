[constructible_from<T, Args...>]()，当可以使用_Args_类型参数初始化_T_类型对象时，满足概念。其基本实现如下：

```cpp
template<typename T, typename ... Args>
concept constructible_from = std::destructible<T> && std::is_convertible_v<T, Args...>;
```

