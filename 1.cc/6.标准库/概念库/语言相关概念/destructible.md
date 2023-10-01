[destructible<T\>]()，当_T_类型对象在生存期结束后，能安全销毁时，满足概念。其基本实现如下：

```cpp
template<typename T>
concept destructible = std::is_nothrow_destructible_v<T>;
```

