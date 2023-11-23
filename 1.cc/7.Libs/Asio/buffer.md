[toc]

# const_buffer & mutable_buffer

`asio::const_buffer`和`asio::mutable_buffer`提供安全的缓冲区表示，其不拥有数据的所有权，只维护缓冲区指针和缓冲区尺寸。

`.data()`，获取缓冲区指针。

`.size()`，获取缓冲区尺寸。

# basic_streambuf

`asio::basic_streambuf`基于`std::streambuf`的可自动调整大小的缓冲区。

```cpp
namespace asio {
template <typename Allocator = std::allocator<char>>
class basic_streambuf : public std::streambuf, private noncopyable {};

using streambuf = basic_streambuf<>;
}  // namespace asio
```

