nginx 提供了堆内存分配和释放的封装函数，拓展了日志记录的支持。

- `ngx_alloc(size, log)`，`malloc()` 的封装。

- `ngx_calloc(size, log)`，内存初始化为 0。
