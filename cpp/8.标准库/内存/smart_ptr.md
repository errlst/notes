#### unique_ptr

`unique_ptr` 是独占式智能指针。其内部通常只需要维护一个指针成员和一个删除器成员。

###### 构造

除了直接通过构造函数构造 `unique_ptr` 对象，还可以使用 `make_unique()` 函数簇，但此时无法指定删除器。

###### 访问

`unique_ptr` 的行为和裸指针一致。也可通过 `get()` 获取当前管理的指针。

###### 修改

`.release()`，返回当前管理的指针，且释放所有权。

`.reset(p)`，重置管理对象，并返回之前管理的对象。

#### shared_ptr & weak_ptr

`shared_ptr` 和 `weak_ptr` 适用于共享情况的指针管理，其通过内存控制块对资源进行管理。

###### 内存控制块

内存控制块通常实现会包含以下数据成员：

- 指针。
- 共享次数。
- 弱引用次数。
- 删除器（当共享次数和弱引用次数均归零后，通过删除器释放管理的指针）。
- 分配器（分配器负责控制内存控制块自身的分配和释放）。

#### enable_shared_from_this

必须使用 public 继承，否则会导致无法正常初始化 weak_ptr，调用 `shared_from_this()` 时抛出异常。

若 _T_ 继承自 `enable_shared_from_this<T>`，若使用智能指针管理 _T_ 对象，可以安全获取其自身的共享指针，而不会导致重复析构的问题。

```cpp
class T : public std::enable_shared_from_this<T> {
public:
    T() = default;

    auto share() -> std::shared_ptr<T> {
        return shared_from_this();
    }
};

auto good_way() {
    auto p = std::shared_ptr<T>{};
    auto p2 = p->share();
}

auto bad_way1() {
    auto t = T{};
    auto p = t.share();
}

auto bad_way2() {
    auto p = std::shared_ptr<T>{};
    auto p2 = std::shared_ptr<T>{ p.get() };
}
```

在构造函数内部不能使用 `shared_from_this()`，因为此时还未进行 `shared_ptr` 的构造，`enable_shared_from_this` 中的数据还未进行有效初始化。
