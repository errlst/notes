<link href="../../style.css" rel="stylesheet">

sender/receiver 模型由四个主要部分构成：

- scheduler，计算资源的轻量句柄，可以是线程、线程池、GPU 等。

- sender，延迟计算。

- operation_state，封装异步操作。

- receiver，接受异步操作的结果。

这四个部分可以简单描述为如下 concept：

```cpp
concept scheduler:
  schedule(scheduler) -> sender;

concept sender:
  connect(sender, receiver) -> operation_state;

concept operation_state:
  start(operation_state) -> void;

concept receiver:
  set_value(receiver, Values...) -> void;
  set_error(receiver, Error) -> void;
  set_stopped(receiver) -> void;
```

## 实现一个 async_read

<div class="code_block">

<div>
operation_state:

```cpp
struct immoveable {
  immoveable() = default;
  immoveable(immoveable &&) = delete;
};

template <receiver Receiver>
struct async_read_operation : public immoveable {
  async_read_operation(asio::ip::tcp::socket *sock, Receiver &&rec)
      : sock_{sock}, rec_{std::move(rec)} {}

  auto start() noexcept {
    sock_->async_read_some(asio::buffer(buf_, sizeof(buf_)), [this](asio::error_code ec, size_t n) {
      if (ec) {
        set_error(std::move(rec_), ec);
      } else {
        set_value(std::move(rec_), vector<char>((char *)buf_, (char *)buf_ + n));
      }
    });
  }

private:
  asio::ip::tcp::socket *sock_;
  Receiver rec_;
  char buf_[1024];
};

```

</div>

<div>
sender:

```cpp
struct async_read_sender {
  using sender_concept = sender_t;

  /* 声明异步操作的完成方式 */
  using completion_signatures =
      completion_signatures<
          set_value_t(std::vector<char>),
          set_error_t(asio::error_code)>;

  async_read_sender(asio::ip::tcp::socket *sock) : sock_{sock} {}

  auto connect(receiver auto rec) -> decltype(auto) {
    return async_read_operation{sock_, std::move(rec)};
  }

private:
  asio::ip::tcp::socket *sock_;
};
```

</div>
</div>

<div class = "code_block">
<div>
usage:

```cpp
auto async_read(asio::ip::tcp::socket *sock) -> async_read_sender {
  return {sock};
}

auto main() -> int {
  auto io = asio::io_context{};
  auto guard = asio::make_work_guard(io);

  std::thread{[&io] {
    auto acceptor = asio::ip::tcp::acceptor{io, asio::ip::tcp::endpoint{asio::ip::tcp::v4(), 8888}};
    acceptor.listen();

    auto sock = std::make_shared<asio::ip::tcp::socket>(acceptor.accept());
    println("recv client");

    while (true) {
      sender auto s = async_read(sock.get());
      try {
        println("recv {}", sync_wait(s).value());
      } catch (...) {
        println("client disconnect");
        return;
      }
    }
  }}.detach();

  return io.run();
}
```

</div>
</div>

