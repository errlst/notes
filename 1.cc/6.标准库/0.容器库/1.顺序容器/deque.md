[deque]()是支持随机访问的双端队列容器，标准库通常使用不连续分配的内存存放元素，libstdc++中的结构大致如下：

```cpp
template<typename T, typename Alloc = std::allocator<T>>
class deque {
    struct Iterator {
        T *m_cur;   // 当前元素位置
        T *m_first; // 当前内存块起始位置
        T *m_last;  // 当前内存块结束位置
        T **m_node; // 当前内存块在内存表中的位置
    };

    T **m_map;          // 管理不连续内存的表
    size_t m_map_size;  // 表大小
    Iterator m_start;   // 首元素
    Iterator m_end;     // 尾巴元素
};
```

因为[deque]()的不连续分配内存结构，扩张时相比[vector]()效率更高[^1]，但在访问元素时，需要二次指针解引用。

## 失效

在[deque]()两端插入元素、删除元素，或调整尺寸时，不会使『未擦除的元素』的引用或迭代器失效。[示例](#示例1)

# 示例

## 示例1

```cpp
auto main() -> int {

    auto vec = std::vector{0};
    auto v = vec.begin();
    vec.resize(1000);
    std::cout << *v << "\n";    // 无效内容

    auto deq = std::deque{0};
    auto d = deq.begin();
    deq.resize(1000);
    std::cout << *d << "\n";    // 0

    return 0;
}
```

[^1]:不需要复制元素到新内存位置，只需要将新分配的内存链接到表中