[same_as<T, U>]()，当_T_和_U_表示同一类型时，满足概念。其基本实现如下：

```cpp
template<typename T, typename U>
concept same_as = std::is_same_v<T, U> && std::is_same_v<U, T>;
```

进行两次类型检查，是为了保证[same_as<T, U>]()蕴含[same_as<U, T>]()，反之亦然。

