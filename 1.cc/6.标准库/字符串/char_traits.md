[char_traits<CharT\>]()是特性类模板，对给定的字符类型提供基础操作。

标准库特化了[char]()、[wchar_t]()、[char8_t]()、[char16_t]()和[char32_t]()，且均满足[字符特性]()要求。

## 字符特性

字符特性要求以下静态成员函数：

* [assign(dst, src)]()，将字符_src_赋值给_dst_。
* [move(dst, src, count)]()、[copy(dst, src, count)]()，移动、拷贝字符序列。
* [eq(lhs, rhs)]()、[lt(lhs, rhs)]()，比较两个字符。
* [compare(lhs, rhs, count)]()，比较两个字符序列。
* [length(str)]()，返回字符序列长度。
* [find(str, count, ch)]()，字符序列中查找字符。
* [eof()]()，返回不等价任何[CharT]()合法值的值。
* [not_eof(ch)]()，检查_ch_是否合法。

## 实现一个大小写无关的字符特性

```cpp
struct case_ignored_char_trait : public std::char_traits<char> {
    static auto eq(char lhs, char rhs) -> bool {
        return std::toupper(lhs) == std::toupper(rhs);
    }

    static auto le(char lhs, char rhs) -> bool {
        return std::toupper(lhs) < std::toupper(rhs);
    }

    static auto compare(const char* lhs, const char* rhs, size_t count) -> int {
        while (--count != 0) {
            if (std::toupper(*lhs) < std::toupper(*rhs)) {
                return -1;
            } else if (std::toupper(*lhs) > std::toupper(*rhs)) {
                return 1;
            }
            ++lhs, ++rhs;
        }
        return 0;
    }
};

auto main() -> int {
    using case_ignored_string = std::basic_string<char, case_ignored_char_trait>;
    auto str1                 = case_ignored_string{"hello"};
    auto str2                 = case_ignored_string{"HELLO"};
    std::cout << (str1 == str2) << "\n";  // true

    return 0;
}
```



