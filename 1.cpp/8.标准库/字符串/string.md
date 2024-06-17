`basic_string<CharT, Traits, Alloc>` 连续存放的字符序列容器。对于短字符串，存在 SSO 优化。以下使用 `string` 指代 `basic_string`。

```cpp
// gcc 实现
struct string {
    auto is_large() -> bool { return ptr_ != buf_; }
    auto data() -> char * { return ptr_; }
    auto size() -> size_t { return size_; }
    auto capacity() -> size_t { return cap_; }

    char *ptr_;
    size_t size_;
    union {
        size_t cap_;
        char buf_[16];
    };
};
```

#### 常量求值

`string` 的成员函数是 `constexpr` 的，因此可以在常量表达式中创建并使用 `string` 对象。

但不能创建 `constexpr` 的 `string` 对象，因为动态分配的存储必须在同一常量表达式中释放。

```cpp
auto main() -> int {
    constexpr auto i = std::string{"hello"}.size();	// ok
    constexpr auto s = std::string{"hello"};  		// error

    return 0;
}
```

#### 转换函数

`stoi(s, pos, base)`、`stol(s, pos, base)`、`stoll(s, pos,base)`，将字符串转换为有符号整数：

* 使用 `isspace()` 舍弃起始的所有非空白符。
* 如果提供 _pos_ ，其接受首个未转换字符的索引。
* 如果 _base_ 为 0，将自动检测数值进制，进制的合法范围为 0~36。

同上，还有 `stoul()`、`stof()`、`stod()`、`stold()` 可以将字符串转换为无符号整数、浮点数。

