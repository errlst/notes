[any]()可以存放任意类型的单个对象的安全容器，对于较小类型对象[^1]，直接存放在内部缓冲区中，对于较大类型对象，存放在动态内存中。其存储结构实现如下：[示例](#示例1)

```cpp
union any_storage {
    void *ptr;
    char buf[sizeof(void *)];
};
```

[any]()并非模板类，但其构造函数和[operator=]()存在模板重载，使其可以存储不同类型对象。

[.has_value()]()检查对象是否存储值，[.type()]()获取当前存储值的类型信息。

## 访问

使用[std::any_cast<Type\>(a)]()对[any]()对象进行安全访问，如果当前存储类型和目标类型不匹配，抛出异常。

# 示例

## 示例1

```cpp
struct small {
    int data[2];
    auto operator new(size_t size) noexcept -> void * {
        std::cout << "small new\n";
        return ::operator new(size);
    }
};

struct big {
    int data[3];
    auto operator new(size_t size) noexcept -> void * {
        std::cout << "big new\n";
        return ::operator new(size);
    }
};

auto main() -> int {
    auto any1 = std::any{small{}};
    auto any2 = std::any{big{}};    // big new

    return 0;
}
```



[^1]:通常为指针大小

