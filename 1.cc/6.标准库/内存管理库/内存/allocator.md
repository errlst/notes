[allocator<T\>]()是标准库提供的默认内存分配器。除了析构函数，[allocator]()的成员函数均是线程安全的。

[.allocate(n)]()，分配可以存放_n_个_T_类型对象的内存空间，不会进行初始化。通过[::operator new(size_t)]()进行内存分配。

[.allocate_at_least(n)]()，分配至少可存放_n_个_T_类型对象的内存空间，不会进行初始化。返回特殊结构体，包含首元素地址和元素数量。

[.deallocate(ptr, n)]()，解分配获取的内存空间，不会进行析构。通过[::operator delete(size_t)]()进行内存释放。

# 示例

```cpp
auto main() -> int {
    struct T {
        T() { std::cout << "construct\n"; }
        ~T() { std::cout << "destruct\n"; }
        static auto operator new(size_t) noexcept -> void * { return nullptr; }
        static auto operator delete(void *, size_t) noexcept -> void {}
    };
    auto alloc = std::allocator<T>{};
    auto p = alloc.allocate(1);
    alloc.deallocate(p, 1);
   
    return 0;
}
// 无输出
```



