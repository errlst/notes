[toc]

[basic_string<CharT, Traits, Alloc>]()是连续存放的字符序列容器。

```cpp
template <typename CharT,
          typename Traits = std::char_traits<CharT>,
          typename Alloc  = std::allocator<CharT>>
class basic_string {
    // 空基类优化，避免单独创建Alloc对象，占用不必要的空间
    struct AllocHide : public Alloc {
        CharT* m_ptr;
    };

  private:
    AllocHide m_data;
    size_t    m_len;
    union {
        size_t m_capacity;
        CharT  m_local_storage[15 / sizeof(CharT) + 1];  // 短字符串直接存放在静态内存中
    };
};
```

## 常量求值

[basic_string]()的成员函数是[constexpr]()的，因此可以在常量表达式中创建并使用[basic_string]()对象；但不能创建[constexpr]()的[basic_string]()对象，因为动态分配的存储必须在同一常量表达式中释放。[示例](#示例1)

## 成员函数

### 访问函数

[.at(idx)]()，进行有边界检查的元素访问。

[.operator[](idx)]()，无边界检查的元素访问。

[.front()]()，访问首元素。

[.back()]()，访问尾元素。

[.data()]()，返回首元素指针。

### 容量函数

[.empty()]()，检测是否为空字符串。

[.size()]()，返回字符串大小(字符数量)。

[.resize(new_size[, ch])]()，重置字符串大小。

[.capacity()]()，返回当前容量。

[.reserve(new_cap)]()，保证容器至少拥有_new_cap_的容量。

[.shrink_to_fit()]()，调整到合适的容量。

### 搜索函数

[.find(...)]()、[.rfind(...)]()，查找子串首次、最后出现的位置。

[.find_first_of(...)]()、[.find_first_not_of(...)]()，查找字符首次出现、缺失的位置。

[.find_last_of(...)]()、[.find_last_not_of(...)]()，查找字符最后出现、缺失的位置。

[.start_with(s)]()、[.end_with(s)]()，检测字符串是否以_s_为前缀、后缀。

[.contains(s)]()，检测字符串是否包含_s_子串。

## 全局函数

### 数值转换

[stoi(s[, pos, base])]()、[stol(s[, pos, base])]()、[stoll(s[, pos, base])]()，将字符串转换为有符号整数：[示例](#示例2)

* 使用[isspace()]()舍弃起始的所有非空白符。
* 如果提供_pos_指针，其接受首个未转换字符的索引。
* 如果_base_为0，将自动检测数值进制，进制的合法范围为0~36。

[stoul(s[, pos, base])]()、[stoull(s[, pos, base])]()，将字符串转换为无符号整数。

[stof(s[, pos])]()、[stod(s[, pos])]()、[stold(s[, pos])]()，将字符串转换为浮点数。可以接受十进制表示、十六进制表示、科学记数表示、无穷表示、非数表示。

[to_string(v)]()，将数值转换为字符串表示，如同[std::format("{}", v)]()。

# 示例

## 示例1

```cpp
auto main() -> int {
    constexpr auto i = std::string{"hello"}.size();	// ok
    constexpr auto s = std::string{"hello"};  		// error

    return 0;
}
```

## 示例2

```cpp
auto main() -> int {
    size_t pos;
    std::stoi("\t\t123  ", &pos);
    std::cout << pos << "\n";  // 5

    return 0;
}
```

