c++ 提供的协程是无栈的：通过返回到协程调用方暂停协程，且恢复协程所需的数据和栈分离存储。

协程是特殊的函数，在函数定义中，包含 `co_await`、`co_yield` 或 `co_return` 表达式即被编译器标记为协程。

#### 相关对象

每个协程都会关联三个对象：

* _承诺对象_，协程内部访问。协程通过该对象返回结果或异常。

* _协程句柄_，协程外部访问。用于恢复协程执行或销毁协程的无所有权句柄。

* _协程状态_，动态存储分配的内部对象。

#### promise_type

编译器通过 `coroutine_traits` 从协程的返回类型确定 _promise_type_，假定 `R` 为协程的非空返回类型，其非特化的确定表达式为 `R::promise_type`。

因此，如果需要定义用于协程的返回类型 _R_，要么声明 `R::promise_type`，要么特化 `coroutine_traits<R>::promise_type`。

```cpp

template <typename T>
struct task {
    struct promise_type {
        auto get_return_object() -> task<T> {
            return {std::coroutine_handle<promise_type>::from_promise(*this)};
        }

        auto yield_value(T value) -> std::suspend_always {
            val_ = std::forward<T>(value);
            return {};
        }

        auto initial_suspend() -> std::suspend_never { return {}; }

        auto final_suspend() noexcept -> std::suspend_always { return {}; }

        auto unhandled_exception() -> void {}

        T val_;
    };

    std::coroutine_handle<promise_type> coro_;
};

auto range(int n) -> task<int> {
    for (auto i = 0; i < n; ++i) {
        co_yield i;
    }
}

auto main() -> int {
    auto co = range(10);
    while (!co.coro_.done()) {
        std::cout << co.coro_.promise().val_ << "\n";
        co.coro_.resume();
    }

    return 0;
}
```

#### 初始化步骤

1. 使用 `operator new` 分配协程状态对象。

2. 将函数形参复制到协程状态中，按值传递的形参通过移动或复制，按引用传递的保持为引用。

3. 调用 _promise_type_ 的构造函数。如果其拥有匹配协程形参的构造函数，调用该构造函数；否则，调用默认构造函数。

4. 保存 `promise_type.get_return_object()` 的结果，在协程首次暂停时，返回给调用方。自此并包含该步骤为止，抛出的所有异常均传播给调用方。

5. 执行 `co_await promise_type.initial_suspend()`。并等待恢复协程状态。

#### co_return

当协程抵达 `co_return` 时：（离开协程控制流等价 `co_return`）

1. 调用 `promise_type.return_void()`，如果 `co_return` 表达式为空类型；否则，调用 `promise_type.return_value(expr)`。

2. 以创建顺序逆序销毁具有自动存储期的变量。

3. 执行 `co_await promise_type.final_suspend()`。

#### 异常

如果协程因为未捕获异常而终止时：

1. 捕获异常并在处理块中调用 `promise_type.unhandled_exception()`。
2. 执行 `co_await promise_type.final_suspend()`。如果此时恢复协程，行为未定义。

#### co_yield

`co_yield expr` 等价于 `co_await promise_type.yield_value(expr)`。
