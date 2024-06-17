```cpp
auto session(asio::ip::tcp::socket sock) -> void {
    try {
        auto buf = std::array<char, 1024>{};
        while (true) {
            auto n = sock.read_some(asio::buffer(buf));
            sock.write_some(asio::buffer(buf, n));
        }
    } catch (std::exception &e) {
        std::cout << e.what() << "\n";
    }
}

auto listen(asio::io_context &context) {
    auto acceptor = asio::ip::tcp::acceptor{context, {asio::ip::tcp::v4(), 12345}};
    while (true) {
        std::thread{session, acceptor.accept()}.detach();
    }
}

auto main() -> int {
    auto context = asio::io_context{};
    listen(context);
    context.run();
    return 0;
}
```

###### buffer

`buffer()`，将现有的内存封装为提供 asio 使用的缓冲区对象。可以封装不同类型的对象，包括原始指针、连续容器（`vector`、`array`、`string` 等）。