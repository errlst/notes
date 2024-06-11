## async
`async([policy, ]f, args)`，异步运行可调用对象 _f_，并返回保佑调用结果的 `future` 对象。

#### 启动策略
`launch::async`，异步求值，在新的线程上调用 _f_。

`launch::deferred`，惰性求值，在返回的 `future` 对象上首次调用等待函数时，在当前线程中进行同步求值。当前线程和最初调用 `async()` 的线程不一定相同。

```cpp
static auto main_id = std::this_thread::get_id();

auto main() -> int {
    std::async(std::launch::async,
        [] {std::cout << (std::this_thread::get_id() == main_id) << "\n"; })
        .wait();    // false
    std::async(std::launch::deferred,
        [] {std::cout << (std::this_thread::get_id() == main_id) << "\n"; })
        .wait();    // true
    return 0;
}
```

在不显示指定启动策略时，其策略为 `launch::async | launch::deferred`，即当无法利用更多并发性时，进行惰性调用。

#### tips
如果 `async()` 返回值被忽略，那么在表达式结尾，`future` 的析构函数会阻塞等待异步求值完成。
```cpp
auto main() -> int {
    std::async([] {
        std::this_thread::sleep_for(std::chrono::seconds{ 1 });
        std::cout << "first step\n";
        });                                 // 1
    std::cout << "second step\n";           // 2
    auto f = std::async([] {
        std::this_thread::sleep_for(std::chrono::seconds{ 1 });
        std::cout << "third step\n";        // 4
        });
    std::cout << "final step\n";            // 3
    f.wait();
    return 0;
}
```

且只有通过 `async()` 获取的 `future` 对象，在析构函数时才可能会阻塞。

## future
`future` 存储异步操作返回的结果，或抛出的异常。

#### 访问
`future` 存放的内容只能被访问一次，如果需要多次访问，可使用 `shared_promise`。

`.get()`，阻塞获取 `future` 存储的结果，或抛出异常。

#### 状态
`.valid()`，检查当前是否还具有共享状态。对非共享的 `future` 进行修改或访问，都是未定义行为。

```cpp
auto main() -> int {
    auto pro = std::promise<int>{};
    auto fut = pro.get_future();
    std::cout << fut.valid() << "\n";  // true
    pro.set_value(0);
    std::cout << fut.valid() << "\n";  // true
    fut.get();
    std::cout << fut.valid() << "\n";  // false
    return 0;
}
```

`.wait()`，等待结果可用。

#### 析构
析构时，如果以下条件均为真，将阻塞：
* 共享状态由 `async()` 创建。
* 共享状态未就绪。
* 当前对象是到其共享状态的唯一引用（对于 `future`，此条件永远为真）。

## promise
`std::promise<T>`内部维护一个`future`，可以通过其访问`future`，也可以设置`future`的值。

## 获取结果

`.get_future()`，返回其关联的`promise`对象。

## 设置结果

`.set_value()`，设置`future`的值，并使其处于就绪状态。

`.set_value_at_thread_exit()`，同上，但当线程退出后才使其处于就绪状态。

`.set_exception()`，设置`future`为异常。

`.set_exception_at_thread_exit()`，同上。

# packaged_task

`packaged_task<R, ...Args>`包装任何可调用对象，使其可以异步调用，并将其返回值或抛出的异常映射到其内部绑定的`future`上。

## 执行

`.operator(args...)`，执行包装的可调用对象。

`.make_ready_at_thread_exit()`，确保只有当线程退出时，`future`才处于就绪状态。

`.reset()`，重新构造`future`状态。

## 结果

`.get_future()`，获取关联的`future`对象。









