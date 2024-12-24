# ngx_buf_t

nginx 为输入、输出操作提供了 `ngx_buf_t` 类型，其用于保存写入目标的数据或从源读取的数据。

缓冲区的内存是单独分配的，和 `ngx_buf_t` 结构无关。缓冲区可以引用内存或文件中的数据，且 `ngx_buf_t` 可以同时引用内存中的数据和文件中的数据。

```c
struct ngx_buf_s {
    // 有效数据范围，[pos, last)，[start, end) 的子范围
    u_char *pos;
    u_char *last;

    // 文件指针 [file_pos, file_last)
    off_t file_pos;
    off_t file_last;

    // 内存块边界，范围 [start, end)
    u_char *start;
    u_char *end;

    ngx_buf_tag_t tag; // 区分缓冲区的唯一值，可以用于缓冲区复用
    ngx_file_t *file;  // 文件对象
    ngx_buf_t *shadow; // 实现多个 ngx_buf_t 共享一个 ngx_buf_t 的数据

    unsigned temporary : 1;     // 缓冲区数据是否可以修改
    unsigned memory : 1;        // 缓冲区引用只读内存
    unsigned mmap : 1;          // 缓冲区内容是 mmap 映射的，不能修改
    unsigned recycled : 1;      // 缓冲区可复用
    unsigned in_file : 1;       // 缓冲区引用文件数据
    unsigned flush : 1;         // 缓冲区数据需要刷新
    unsigned sync : 1;          // 指示缓冲区不带数据或特殊标志（如 flush
    unsigned last_buf : 1;      // 指示缓冲区是最后一个输出缓冲区
    unsigned last_in_chain : 1; // 指示缓冲区是链表中的最后一个缓冲区
    unsigned last_shadow : 1;   // 指示缓冲区是最后一个引用 shadow 的缓冲区
    unsigned temp_file : 1;     // 缓冲区位于临时文件

    int num; // 占位
};
```

## 创建

`ngx_alloc_buf(pool)`，创建一个 `ngx_buf_t`。

```c
#define ngx_alloc_buf(pool) ngx_palloc(pool, sizeof(ngx_buf_t))
```

`ngx_create_temp_buf(pool, size)`，创建一个临时缓冲区。

# ngx_chain_t

对于输出和输出操作，缓冲区会以链的形式使用。

```c
struct ngx_chain_s {
    ngx_buf_t *buf;
    ngx_chain_t *next;
};
```

## 创建

`ngx_alloc_chain_link(pool)`，在 pool 的 chain 链末尾分配一个链节点并返回。
