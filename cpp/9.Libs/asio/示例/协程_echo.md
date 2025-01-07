```cpp
auto echo(asio::ip::tcp::socket sock) -> asio::awaitable<void> {
    try {
        auto buf = std::array<char, 1024>{};
        while (true) {
            auto n = co_await sock.async_read_some(asio::buffer(buf), asio::use_awaitable);
            co_await sock.async_write_some(asio::buffer(buf, n), asio::use_awaitable);
        }
    } catch (std::exception &e) {
        std::cout << e.what() << "\n";
    }
}

auto listen() -> asio::awaitable<void> {
    auto acceptor = asio::ip::tcp::acceptor{co_await asio::this_coro::executor,
                                            {asio::ip::tcp::v4(), 12345}};
    while (true) {
        auto sock = co_await acceptor.async_accept(asio::use_awaitable);
        asio::co_spawn(co_await asio::this_coro::executor, echo(std::move(sock)),
                       asio::detached);
    }
}

auto main() -> int {
    try {
        auto context = asio::io_context{};
        asio::co_spawn(context, listen(), asio::detached);
        context.run();
    } catch (std::exception &e) {
        std::cout << e.what() << "\n";
    }
    return 0;
}
```

###### awaitable

`awaitable<T, Executor>` 用于协程返回的类型。`awaitable` 的 _promise_type_ 声明如下：

```cpp
namespace std{
    template <typename T, typename Executor, typename... Args>
    struct coroutine_traits<asio::awaitable<T, Executor>, Args...> {
        typedef asio::detail::awaitable_frame<T, Executor> promise_type;
    };   
}
```

###### token

asio 异步模型是通过 _completion token_ 支持多种组合机制的，约定所有异步操作的最后一个参数为接受标记。

* 传递函数对象，异步操作结束后，将结果传递给函数对象然后调用。

* 传递 `use_future`，异步函数返回用于等待结果的 `future` 对象。
* 传递 `use_awaitable`，异步函数的行为类似于基于 `awaitable` 的协程，此时异步函数不会直接启动，而是返回 `awaitable`。
* 传递 `detached`，异步操作被分离。
* 传递 `deferred`，异步操作返回函数对象以延迟启动。

###### use_awaitable

对于需要用于协程的异步函数，需要使用 `use_awaitable` 标记。

###### this_coro::executor

`this_coro::executor` 是返回当前协程执行器的 _awaitable_ 对象。

###### co_spawn

`co_spawn(executor, awaitable)` 生成一个基于协程的执行线程。

* _awaitable_ 可以是 `awaitable` 对象，也可以是返回 `awaitable` 的函数对象。

