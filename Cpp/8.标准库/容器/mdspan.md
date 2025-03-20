`mdspan<T, Extents, Layout, Access>` 是将多维索映射到具体元素的多维数组视图，映射和元素的访问策略是可配置的，底层数组可以不连续，甚至不需要存在于内存。

- Extends，维度的数量、大小，必须是 `std::extents` 的特化。

- Layout，将多维索转为一维索引的策略。

- Access，使用一维索引访问元素的策略。



