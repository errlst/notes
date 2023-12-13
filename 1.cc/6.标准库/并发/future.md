[toc]

# future

`std::future<T>`存储异步操作返回的结果，或抛出的异常。通常通过`std::async()`、`std::packaged_task`获取绑定到异步流程的`future`对象。

## 访问

`future`存放的内容只能被访问一次，如果需要多次访问，可使用`std::shared_promise`。

`.get()`，阻塞获取`future`存储的结果，或抛出异常。

## 状态

`.valid()`，检查当前是否还具有共享状态。对非共享的`future`进行修改或访问，都是未定义行为。

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

## 析构

`future`对象析构时，如果其状态未就绪，且满足以下条件，其会阻塞直：

* 对象由`async()`创建。
* 共享状态未就绪。
* 当前对象持有共享状态的唯一引用。

# promise

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

# async

`async([policy,] f, args...)`，异步执行可调用对象，并返回`future`对象，存放可调用对象的返回值或异常。

## 策略

`std::launch::async`，运行新线程，进行异步求值。

`std::launch::deferred`，首次请求结果时开始执行任务。

## TIPS

如果`async()`的返回值被忽略，那么在表达式结尾，`future`的析构函数会阻塞直到异步计算完成。

```cpp
auto main() -> int {
    std::async([] {
        std::this_thread::sleep_for(std::chrono::seconds{1});
        std::cout << "first\n";
    });  // 阻塞直到异步任务完成后，才继续往下执行
    std::async([] {
        std::cout << "second\n";
    });

    return 0;
}  // first + second
```







