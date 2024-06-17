#### mdspan

`mdspan<T, Extends, LayoutPolicy, AccessPolicy>` 是对到对象的连续序列的视图，并将其重新解析为多维数组。

###### 构造

`mdspan(data, exts...)` 将对象序列以 _exts..._ 的形式重新解析。

```cpp
auto main() -> int {
    auto buf = std::array<int, 4>{1, 2, 3, 4};
    auto mdsp = std::mdspan{buf.data(), 2, 2};
    for (auto i = 0uz; i < mdsp.extent(0); ++i) {
        for (auto j = 0uz; j < mdsp.extent(1); ++j) {
            std::print("{} ", mdsp[i, j]);
        }
        std::println("");
    }
    return 0;
}
```





