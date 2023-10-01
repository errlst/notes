[predicate<F, Args...>]()，当_F_满足二元谓词类型时，满足约束。其基本实现如下：

```cpp
template<typename T>
concept boolean_testable_impl = std::convertible_to<T, bool>;

template<typename T>
concept boolean_testable = boolean_testable_impl<T> && requires(T &&t){
    { !std::forward<T>(t) } -> boolean_testable_impl;
};

template<typename F, typename ... Args>
concept predicate = std::regular_invocable<F, Args...>
                    && boolean_testable<std::invoke_result_t<F, Args...>>;
```

