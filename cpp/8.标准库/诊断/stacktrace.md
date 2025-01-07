`basic_stacktrace<Allocator>`，整个函数调用栈的调用痕迹。

#### 构造

`stacktrace::current(skip, max_depth)` 获取当前调用栈的 `stacktrace` 对象。

* _skip_，可选参数，需要跳过的栈踪条目数量。
* _max_depth_，可选参数，栈踪最大条目数量。

#### stacktrace_entry

`stacktrace_entry` 是具体的单个栈踪条目。