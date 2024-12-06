`any` 可以存放任意类型的单个对象的安全容器，对于较小类型对象，直接存放在内部缓冲区中，对于较大类型对象，存放在动态内存中。其逻辑结构类似：

```cpp
class any{
    union any_storage {
        void *heap_mem_;
        char stack_mem_[sizeof(heap_mem_)];
    };
    _any_manager_t manager_;    // 通过类型擦除操作数据
}
```

`any` 非模板类，但其构造函数和 `operator=()` 定义为模板，且其内部进行了类型擦除，因此可以存储不同类型对象。

# 访问

`.has_value()` 检查是否存储对象。

`.type()` 获取存储对象的类型信息。

`std::any_cast<T>(a)` 进行安全访问 `any` 对象，如果当前存储类型和目标类型不匹配，抛出异常。
