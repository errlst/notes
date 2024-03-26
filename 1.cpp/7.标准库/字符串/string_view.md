[basic_string_view<CharT, Traits>]()，描述一个能指代常量连续访问字符对象序列的对象，其典型实现只保留两个成员：指向字符序列的指针和字符序列的大小：

```cpp
template <typename CharT, typename Traits = std::char_traits<CharT>>
class basic_string_view {
  private:
    size_t       m_len;
    const CharT* m_str;
};
```

[basic_string_view]()的许多操作和[basic_string]()一致。

## 字面量

[basic_string_view]()可以作为编译期常量，标准库还提供了[operator""sv()]()重载，以定义[basic_string_view]()字面量。



