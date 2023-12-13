[toc]

# unique_ptr

`std::unique_ptr<T, Deleter>` 是独占式智能指针。其内部通常只需要维护一个指针成员和一个删除器成员。

## 构造

除了直接通过构造函数构造 `unique_ptr` 对象，还可以使用 `std::make_unique()` 函数簇。通过 `make_unique()` 函数簇创建 `unique_ptr` 对象时，只能使用默认删除器。

> * `make_unique<T>(args...)`，构造管理单个对象的 `unique_ptr` 对象。
> * `make_unique<T>(size)`，构造管理数组的 `unique_ptr` 对象，_T_ 是未知边界的数组类型。

## 访问

对于管理单个对象的 `unique_ptr`，重载了 `operator*()` 和 `operator->()`；对于管理数组的 `unique_ptr`，重载了`operator[]()`。使得 `unique_ptr` 的行为和裸指针一致。

`.get()`，返回管理的指针。

`.get_deleter() `，返回删除器对象的引用。

## 修改

`.release()`，返回管理的指针，并释放其所有权。

`.reset(p)`，重置管理对象，并释放之前管理的对象。

## 实现

```cpp
/* 针对管理非数组资源的unique_ptr的简易实现 */
template <typename T>
struct default_deleter {
    constexpr default_deleter() noexcept = default;
    template <typename U>
    constexpr default_deleter(const default_deleter<U>&) noexcept {}
    constexpr ~default_deleter() = default;

  public:
    constexpr auto operator()(T* p) const {
        static_assert(!std::is_void_v<T> && sizeof(T) >= 0, "can't delete incomplete type");
        delete p;
    }
};

template <typename T, typename D = default_deleter<T>>
class unique_ptr {
  private:
    // 萃取pointer的辅助类
    template <typename T_, typename D_NoRef, typename = void>
    struct unique_ptr_pointer {
        using type = T_*;
    };

    template <typename T_, typename D_NoRef>
    struct unique_ptr_pointer<T_, D_NoRef, std::void_t<typename D_NoRef::pointer>> {
        using type = D_NoRef::pointer;
    };

  public:
    using pointer = unique_ptr_pointer<T, std::remove_reference_t<D>>::type;
    using element_type = T;
    using deleter_type = D;

  public:
    constexpr unique_ptr() noexcept : unique_ptr{pointer{}} {}

    constexpr explicit unique_ptr(std::type_identity_t<pointer> p) noexcept : m_ptr{p} {
        static_assert(!std::is_pointer_v<deleter_type>, "deleter can't be pointer");
        static_assert(std::is_default_constructible_v<deleter_type>, "deleter must default constructible");
    }

    constexpr unique_ptr(unique_ptr&& other) noexcept {
        static_assert(std::is_move_assignable_v<D>, "deleter must move assignable");
        if (this != std::addressof(other)) {
            m_ptr = other.release();
            m_deleter = std::forward<D>(other.m_deleter);
        }
    }

    constexpr ~unique_ptr() {
        if (get()) {
            get_deleter()(get());
        }
    }

  public:
    constexpr auto operator=(unique_ptr&& other) noexcept -> unique_ptr& {
        static_assert(std::is_move_assignable_v<D>, "deleter must move assignable");
        if (this != std::addressof(other)) {
            reset(other.release());
            get_deleter() = std::forward<D>(other.get_deleter());
        }
        return *this;
    }

    constexpr auto operator*() const noexcept(noexcept(*std::declval<pointer>())) -> std::add_lvalue_reference_t<T> {
        assert(get() != pointer{});
        return *get();
    }

    constexpr auto operator->() const noexcept {
        assert(get() != pointer{});
        return get();
    }

    constexpr explicit operator bool() const noexcept {
        return get() != pointer{};
    }

  public:
    constexpr auto get() const noexcept -> pointer { return m_ptr; }

    constexpr auto get_deleter() noexcept -> deleter_type& { return m_deleter; }

    constexpr auto get_deleter() const noexcept -> const deleter_type& { return m_deleter; }

    constexpr auto release() noexcept {
        auto ptr = m_ptr;
        m_ptr = pointer{};
        return ptr;
    }

    constexpr auto reset(pointer p = pointer{}) noexcept {
        auto old = get();
        m_ptr = p;
        if (old != pointer{}) {
            get_deleter()(old);
        }
    }

    constexpr auto swap(unique_ptr& other) noexcept {
        static_assert(std::is_swappable_v<D>, "deleter must swappable");
        std::swap(m_ptr, other.m_ptr);
        std::swap(m_deleter, other.m_deleter);
    }

  private:
    pointer m_ptr;
    deleter_type m_deleter;
};

template <typename T, typename... Args>
constexpr auto make_unique(Args&&... args) -> unique_ptr<T> {
    return {new T(std::forward<Args>(args)...)};
}
```

# shared_ptr

`std::shared_ptr<T>` 是共享式智能指针，多个 `shared_ptr` 可以共享一个对象，当最后一个管理当前对象的 `shared_ptr` 析构或释放所有权时，销毁其管理的对象，并释放分配的内存。

