`super_block` 表示一个挂载完成的文件系统。

内核中维护了一个 `super_block` 列表，该列表中包含了所有已挂载的 `super_block` 对象。vfs 查找 `super_block` 时，遍历列表判断 `s_type` 字段和 `file_system_type` 是否相同。

```cpp
// fs/super.c

static LIST_HEAD(super_blocks);
static DEFINE_SPINLOCK(sb_lock);
```
