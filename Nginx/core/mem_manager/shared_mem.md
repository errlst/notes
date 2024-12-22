nginx 通过共享区域在进程间共享数据，将相同的物理内存映射到不同的虚拟地址空间中。

nginx 的共享区域管理在 cycle 上的一个共享区域链表中。

## ngx_shm_zone_t

```c
typedef ngx_int_t (*ngx_shm_zone_init_pt)(ngx_shm_zone_t *zone, void *data);

struct ngx_shm_zone_s {
    void *data;
    ngx_shm_t shm;
    ngx_shm_zone_init_pt init;
    void *tag;
    void *sync;
    ngx_uint_t noreuse;
};
```

- `data`，传递给 `init` 的数据。

- `shm`，平台相关对象。

- `init`，共享区域映射到实际内存后调用。

- `tag`，共享区域标签，一般同模块的共享内存的 tag 相同。

- `sync`，进程同步需要的数据。

- `noreuse`，共享区域是否可重用。

## ngx_shm_t

```c
/// src/os/unix/ngx_shmem.h
typedef struct {
    u_char      *addr;
    size_t       size;
    ngx_str_t    name;
    ngx_log_t   *log;
    ngx_uint_t   exists;
} ngx_shm_t;
```

`ngx_shm_t` 保存特定平台所需数据，但至少拥有以下字段：

- `addr`，映射的共享内存地址。

- `size`，共享区域大小。

- `name`，共享区域名称，在所有模块的所有共享区域中唯一。

- `log`，日志。

## 创建

`ngx_shared_memory_add(conf, name, size, tag)`，添加共享区域，添加到 `conf.cycle.shared_memory` 后面。

> - 如果存在 `name` 相同但 `tag` 不同的共享区域，则报错并返回 null。
>
> - 如果存在 `name` 和 `tag` 都相同的共享区域：
>
>   - 如果 `size` 为 0，则将区域的 size 设为 0，并返回。
>
>   - 如果 `size` 和当前 size 不等，则报错并返回 null。
>
>   - 否则，返回已存在的共享区域。
>
> - 否则，创建新的共享区域并返回。

## ngx_slab_pool_t

nginx 提供 `ngx_slab_pool_t` 在共享区域中分配内存，在初始化 cycle 时，会自动创建一个用于分配的 slab 池，可以通过表达式 `(ngx_slab_pool_t*)(((ngx_shm_zone_t*)cycle->shared_memory.part.elts)->shm.addr)` 访问。

## 示例

```c
void test_shared() {
    ngx_shm_t shm;
    shm.size = sizeof(ngx_int_t);
    ngx_str_set(&shm.name, "test_shm");
    if (ngx_shm_alloc(&shm) != NGX_OK) {
        return;
    }

    ngx_int_t *shared = (ngx_int_t *)shm.addr;
    ngx_int_t *unshared = malloc(sizeof(ngx_int_t));
    *shared = *unshared = 12345;
    printf("parent shared %p: %ld\n", shared, *shared);
    printf("parent unshared %p: %ld\n\n", unshared, *unshared);

    if (fork() == 0) {
        printf("child shared %p: %ld\n", shared, *shared);
        printf("child unshared %p: %ld\n\n", unshared, *unshared);
        sleep(5);
        printf("child shared %p: %ld\n", shared, *shared);
        printf("child unshared %p: %ld\n\n", unshared, *unshared);
        exit(0);
    }
    sleep(2);
    *shared = *unshared = 54321;
    printf("parent waiting\n\n");
    sleep(10);
}
```

```text
parent shared 0x7fc7d8d19000: 12345
parent unshared 0x55716f8722a0: 12345

child shared 0x7fc7d8d19000: 12345
child unshared 0x55716f8722a0: 12345

parent waiting

child shared 0x7fc7d8d19000: 54321
child unshared 0x55716f8722a0: 12345
```
