[toc]

# basic_waitable_timer

`asio::basic_waitable_timer<Cloc>`提供可等待的计时器功能。

```cpp
namespace asio {
template <typename Clock,
          typename WaitTraits = asio::wait_traits<Clock>,
          typename Executor = asio::any_io_executor>
class basic_waitable_timer {};

using system_timer = basic_waitable_timer<std::chrono::system_clock>;
using steady_timer = basic_waitable_timer<std::chrono::steady_clock>;
}  // namespace asio
```

## 构造

`basic_waitable_timer(io)`，构造没有设置任何到期时间的定时器。

`basic_waitable_timer(io, time_point)`，构造到_time_point_时间点到期的定时器。

`basic_waitable_timer(io, duration)`，构造经过_duration_时间段后到期的定时器。

## 等待

`.async_wait(tok)`，异步等待，_tok_需要满足签名`void(asio_error)`。计时器的所有异步操作都会被加入一个异步队列中，定时器未到期之前，异步操作都可以被取消。

> * `.cancel()`，取消计时器上的所有异步操作。
>
> * `.cancel_one()`，取消计时器上的一个异步操作，根据队列特性，先注册的异步操作先被取消。
>
>   ```cpp
>   auto main() -> int {
>       auto io = asio::io_context{};
>       auto timer = asio::steady_timer{io, std::chrono::seconds{1}};
>       auto handle = [](size_t idx, const asio::error_code& err) {
>           if (err) {
>               std::cout << std::format("async{} err\n", idx);
>           } else {
>               std::cout << std::format("async{}\n", idx);
>           }
>       };
>       timer.async_wait(std::bind_front(handle, 1));
>       timer.async_wait(std::bind_front(handle, 2));
>       timer.cancel_one();  // async1 err
>       io.run();            // async2
>       return 0;
>   }
>   ```

`.wait()`，同步等待，返回错误码。

## 定时

重新定时后，会取消计时器上的所有异步操作，并返回取消的异步操作的数量。

`.expires_after(duration)`，定时到一段时间后。

`.expires_at(time_point)`，定时到指定时间点。







