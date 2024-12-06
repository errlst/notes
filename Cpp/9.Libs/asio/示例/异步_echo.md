```cpp
// 通过 shared_from_this，让自身的引用计数可以在异步操作中保持有效，而无需显示维护在某个地方
class session : public std::enable_shared_from_this<session> {
  public:
    session(asio::ip::tcp::socket sock) : sock_(std::move(sock)) {}

    auto start() { do_read(); }

  private:
    auto do_read() -> void {
        sock_.async_read_some(asio::buffer(buf_),
                              [self = shared_from_this()](const asio::error_code &ec, size_t len) {
                                  if (!ec) {
                                      self->do_write(len);
                                  }
                              });
    }

    auto do_write(size_t len) -> void {
        sock_.async_write_some(asio::buffer(buf_, len),
                               [self = shared_from_this()](const asio::error_code &ec, size_t len) {
                                   if (!ec) {
                                       self->do_read();
                                   }
                               });
    }

  private:
    asio::ip::tcp::socket sock_;
    std::array<char, 1024> buf_;
};

class server {
  public:
    server(asio::io_context &context)
        : acceptor_{context, {asio::ip::tcp::v4(), 12345}} { do_accept(); }

  private:
    auto do_accept() -> void {
        acceptor_.async_accept([this](const asio::error_code &ec, asio::ip::tcp::socket sock) {
            if (!ec) {
                std::make_shared<session>(std::move(sock))->start();
            }
            do_accept();
        });
    }

  private:
    asio::ip::tcp::acceptor acceptor_;
};

auto main() -> int {
    auto context = asio::io_context{};
    auto s = server{context};
    context.run();
    return 0;
}
```

asio 的异步操作注册后只会调用一次，因此在异步操作内部，要进行再次注册。