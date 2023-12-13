[toc]

# allocator

`std::allocator<T>`是标准库容器的默认内存分配器，其只是对`operator new`和`operator delete`的简单封装，内部不维护任何状态。

## 分配

`.allocate(n)`，分配`n*sizeof(T)`字节大小的未初始化内存。

`.allocate_at_least(n)`，分配至少可以存放_n_个_T_类型对象的内存空间，返回的结构包含首元素地址和可存放的元素数量。

## 释放

`.deallocate(p, n)`，释放通过`allocator`获取的内存空间，不会进行析构操作。

## 实现

```cpp
template <typename Ptr, typename SizeType = std::size_t>
struct allocation_result {
    Ptr ptr;
    SizeType count;
};

template <typename T>
class allocator {
    static_assert(!std::is_const_v<T> && !std::is_volatile_v<T> && !std::is_function_v<T> && !std::is_reference_v<T>);

  public:
    using value_type = T;
    using size_type = std::size_t;
    using difference_type = std::ptrdiff_t;
    using propagate_on_container_move_assignment = std::true_type;

  public:
    constexpr allocator() noexcept = default;
    constexpr allocator(const allocator&) noexcept = default;
    constexpr auto operator=(const allocator&) -> allocator& = default;
    template <typename U>
    constexpr allocator(const allocator<U>&) noexcept {}
    constexpr ~allocator(){};

  public:
    [[nodiscard]] constexpr auto allocate(size_t n) {
        if (std::numeric_limits<size_type>::max() / sizeof(value_type) < n) {
            throw std::bad_array_new_length{};
        }
        return ::operator new(sizeof(T) * n);
    }
    [[nodiscard]] constexpr auto allocate_at_least(size_t n) -> allocation_result<value_type*> {
        if (std::numeric_limits<size_type>::max() / sizeof(value_type) < n) {
            throw std::bad_array_new_length{};
        }
        return {allocate(n)};
    }
    [[nodiscard]] constexpr auto deallocate(T* p, size_t n) { ::operator delete(p); }
};
```

