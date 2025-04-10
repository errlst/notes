`unique_ptr` 是独占式智能指针。其内部通常只需要维护一个指针成员和一个删除器成员。

### 构造

除了直接通过构造函数构造 `unique_ptr` 对象，还可以使用 `make_unique()` 函数簇，但此时无法指定删除器。

### 访问

`unique_ptr` 的行为和裸指针一致。也可通过 `get()` 获取当前管理的指针。

### 修改

`.release()`，返回当前管理的指针，且释放所有权。

`.reset(p)`，重置管理对象，并返回之前管理的对象。
