## 函数包装

### function

[function<typename T\>]()对所有可调用对象的封装。

### mem_fn

[std::mem_fn(pm)]()生成指向成员函数指针的包装对象，其返回类型未定义，但行为和可调用对象一致。[示例](#示例1)

### bind_front

[std::bind_front(f, args...)]()，返回一个可调用对象，其是对_f_以及_args_的封装。[实现](#实现1)

## 函数调用

### invoke

[std::invoke(f, args...)]()，通过参数调用可调用对象[^1]，以及访问数据成员。[实现](#实现2)



# 示例

## 示例1

```cpp
auto main() -> int {
    struct T {
        auto call() -> void {
            std::cout << "call\n";
        }
    } t;
    auto f1 = std::mem_fn(&T::call);
    f1(t);  // call
    auto f2 = std::function<void(T &)>{&T::call};
    f2(t);  // call

    return 0;
}
```

# 实现

## 实现1

```cpp
template<typename Fn, typename ... BoundArgs>
class BindFront {
  public:
    template<typename Fn_, typename ... BoundArgs_>
    explicit BindFront(Fn_ &&fn, BoundArgs_ &&... args) :
        m_fn{std::forward<Fn_>(fn)}, m_bound_args{std::forward<BoundArgs_>(args)...} {
        static_assert(sizeof...(BoundArgs) == sizeof...(BoundArgs_));
    }

    template<typename ... CallArgs>
    auto operator()(CallArgs &&... call_args) & -> std::invoke_result_t<Fn, BoundArgs..., CallArgs...> {
        return [&]<size_t... Idx>(std::index_sequence<Idx...>) {
            return std::invoke(std::forward<decltype(m_fn) &>(m_fn),
                               std::get<Idx>(std::forward<decltype(m_bound_args) &>(m_bound_args))...,
                               std::forward<CallArgs>(call_args)...);
        }(std::index_sequence_for<BoundArgs...>{});
    }
  private:
    Fn m_fn;
    std::tuple<BoundArgs...> m_bound_args;
};

// 无绑定参数的特化，未实现
template<typename Fn>
class BindFront<Fn> {
  private:
    Fn m_fn;
};

template<typename Fn, typename ... BoundArgs>
auto bind_front(Fn &&fn, BoundArgs &&... args) -> BindFront<Fn, BoundArgs...> {
    return BindFront<std::decay_t<Fn>, std::decay_t<BoundArgs>...>{
        std::forward<Fn>(fn), std::forward<BoundArgs>(args)...};
}
```

## 实现2

```cpp
template<typename Fn, typename T, typename... Args>
auto invoke_mem_fun(Fn &&fn, T &&t, Args &&...args) -> decltype(auto) {
    if constexpr (std::is_pointer_v<std::decay_t<T>>) {
        return (std::forward<T>(t)->*fn)(std::forward<Args>(args)...);
    } else {
        return (std::forward<T>(t).*fn)(std::forward<Args>(args)...);
    }
}

template<typename Mem, typename T>
auto invoke_mem_obj(Mem &&mem, T &&t) -> decltype(auto) {
    if constexpr (std::is_pointer_v<std::decay_t<T>>) {
        return (std::forward<T>(t)->*mem);
    } else {
        return (std::forward<T>(t).*mem);
    }
}

template<typename Fn, typename ... Args>
auto invoke(Fn &&fn, Args &&...args) -> std::invoke_result_t<Fn, Args...> {
    if constexpr (std::is_member_function_pointer_v<std::decay_t<Fn>>) {
        return invoke_mem_fun(std::forward<Fn>(fn), std::forward<Args>(args)...);
    } else if constexpr (std::is_member_object_pointer_v<std::decay_t<Fn>>) {
        return invoke_mem_obj(std::forward<Fn>(fn), std::forward<Args>(args)...);
    } else {
        return fn(std::forward<Args>(args)...);
    }
}
```





[^1]:包括成员函数