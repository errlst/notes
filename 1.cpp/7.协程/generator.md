`generator<Ref, V, Allocator>` 通过反复恢复可返回值的协程以生成元素，每当 `co_yield` 被求值，`generator` 序列就生成一个新的元素。

`generator` 继承了 `view_interface`，其满足视图的行为。

```cpp
auto range(int beg, int end, int step) -> std::generator<int> {
    while (beg < end) {
        co_yield beg;
        beg += step;
    }
}

auto main() -> int {
    for (auto i : range(0, 10, 3)) {
        std::print("{} ", i);
    }
    return 0;
}
```

