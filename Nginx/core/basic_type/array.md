# Array

```c
typedef struct {
    void *elts;
    ngx_uint_t nelts;
    size_t size;
    ngx_uint_t nalloc;
    ngx_pool_t *pool;
} ngx_array_t;
```

- `elts`，指向数组元素。

- `nelts`，数组当前元素数量。

- `size`，单个元素大小。

- `nalloc`，数组所有元素数量。

- `poll`，内存池。

## 创建初始化

- `ngx_array_create(poll, n, size)`，在内存池中创建空的数组对象，并初始化。

- `ngx_array_init(array, pool, n, size)`，使用内存池初始化已创建的数组对象。

- `ngx_array_destroy(array)`，释放数组。

## 添加元素

- `ngx_array_push(array)`，在数组末尾插入一个元素，并返回其指针。

- `ngx_array_push_n(array, n)`，在数组末尾插入 n 个元素，并返回第一个元素的指针。

当当前分配的容量不够添加元素时，如果数组分配在内存池的末尾，且内存池有多余的内存，则直接向后拓展一个元素的内存；否则，在内存池中分配两倍大小的内存块，并复制元素。
