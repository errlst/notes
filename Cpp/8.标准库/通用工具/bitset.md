`bitset<N>` 表示 N 位固定大小的比特序列，可用标准位运算符操作 `bitset`，且可将其与整数或字符串进行相互转换。对位的操作需要通过 `bitset::reference` 间接进行，其实现结构类似：

```cpp
template <size_t N>
class bitset {
  class reference {
    uint32_t *word_ptr_;
    size_t bit_pos_;
  };

  uint32_t bits_[N / (sizeof(uint32_t) * 8) +
                (N % (sizeof(uint32_t) * 8) == 0 ? 0 : 1)];
};
```

# 访问

使用 `operator[](pos)` 获取比特引用，不会进行边界检查。使用 `.test(pos)` 获取比特引用，会进行边界检查。

`.all()`，检查是否所有位都为 1。

`.any()`，检查是否存在位为 1。

`.none()`，检查是否所有位都不为 1。

`.count()`，获取为 1 的比特位数量。

`.size()`，获取比特总数。

# 修改

`.set(pos, v=true)`，设置某个比特位为特定值。

`.reset(pos)`，设置某个比特位为 0。

`.flip(pos)`，翻转某个比特位的值。

# 转换

`.to_string()`，返回比特集的字符串表示。

`.to_ulong()`、`.to_ullong()`，返回比特集的整数表示。
