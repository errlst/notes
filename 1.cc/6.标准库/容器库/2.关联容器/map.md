[map]()是有序键值对容器，其通常实现为红黑树，其声明结构为：

```cpp
template<typename Key, typename Val,
         typename Compare = std::less<Key>,
         typename Alloc = std::allocator<std::pair<const Key, Val>>>
class map;
```

『有序』的意思是：遍历容器中元素时，访问元素的顺序和插入元素时的顺序一致。
