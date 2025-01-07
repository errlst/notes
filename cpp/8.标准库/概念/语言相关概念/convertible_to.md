[convertible_to<From, To>]()，当_From_类型对象可以隐式和显示转换为_To_类型时，满足概念。其基本实现如下：

```cpp
template<typename From, typename To>
concept convertible_to = std::is_convertible_v<From, To> && requires{
    static_cast<To>(std::declval<From>());
};
```

