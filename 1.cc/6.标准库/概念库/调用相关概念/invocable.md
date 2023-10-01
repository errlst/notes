[invocable<F, Args...>]()，当_F_类型对象以及参数可以通过[std::invoke()]()调用时，满足概念。其基本实现如下：

```cpp
template<typename F, typename ... Args>
concept invocable = requires(F &&f, Args &&... args){
    std::invoke(std::forward<F>(f), std::forward<Args>(args)...);
};
```

[regular_invocable<F, Args...>]()增加了语义上的区别：其需要保证传入相同的参数，函数的返回值相同。