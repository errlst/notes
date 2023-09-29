[forward_list]()是支持快速插入、不支持随机访问的单链表容器，其与c语言中实现单链表相比，没有任何空间或时间上的额外开销[^1]。

## 操作

除了基本容器操作，[forward_list]()额外提供了以下操作：

* [.merge(other [, compare])]()，合并两个链表，可额外提供比较器。
* [.splice_after(pos, other [, first, last])]()，移动其他链表的元素到当前链表中。
* [.unique([predicate])]()，移除重复元素，可额外提供比较谓词。

[^1]:[forward_list]()只维护链表的头结点，且其不提供[.size()]()函数。