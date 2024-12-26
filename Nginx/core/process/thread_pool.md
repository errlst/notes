编译时提供 `--with-threads` 后，nginx 可以使用线程池。

目前 nginx 未提供对 windows 平台线程池支持，unix 平台的线程操作是对 pthread 的封装。

```c
/// os/unix/ngx_thread.h
typedef pthread_mutex_t ngx_thread_mutex_t;

ngx_int_t ngx_thread_mutex_create(ngx_thread_mutex_t *mtx, ngx_log_t *log);
ngx_int_t ngx_thread_mutex_destroy(ngx_thread_mutex_t *mtx, ngx_log_t *log);
ngx_int_t ngx_thread_mutex_lock(ngx_thread_mutex_t *mtx, ngx_log_t *log);
ngx_int_t ngx_thread_mutex_unlock(ngx_thread_mutex_t *mtx, ngx_log_t *log);

typedef pthread_cond_t ngx_thread_cond_t;

ngx_int_t ngx_thread_cond_create(ngx_thread_cond_t *cond, ngx_log_t *log);
ngx_int_t ngx_thread_cond_destroy(ngx_thread_cond_t *cond, ngx_log_t *log);
ngx_int_t ngx_thread_cond_signal(ngx_thread_cond_t *cond, ngx_log_t *log);
ngx_int_t ngx_thread_cond_wait(ngx_thread_cond_t *cond, ngx_thread_mutex_t *mtx, ngx_log_t *log);
```

## 使用

- `ngx_thread_pool_get(cf, name)`，获取线程池，如果不存在，返回 null。

- `ngx_thread_pool_add(cf, name)`，获取线程池，如果不存在，新建线程池。

- `ngx_thread_task_post(pool, task)`，添加任务。

> 如果任务已经被加入到线程池（通过 `event.active` 判断），或者任务已满，则添加失败。

## ngx_task_t

```c
struct ngx_thread_task_s {
    ngx_thread_task_t *next;                     // 线程池任务队列链表节点
    ngx_uint_t id;                               // 线程池中的唯一id
    void *ctx;                                   // 自定义数据
    void (*handler)(void *data, ngx_log_t *log); // 线程处理函数，data=ctx
    ngx_event_t event;
};
```

handle 处理完成后，会将 event 注册到 ngxin 的事件驱动中，执行 event->handle。

`ngx_thread_task_alloc(pool, size)` 分配一个任务，`size` 是自定义数据大小。

## ngx_thread_pool_t

```c
/// src/core/ngx_thread_pool.c
struct ngx_thread_pool_s {
    ngx_thread_mutex_t mtx;        // 队列锁
    ngx_thread_pool_queue_t queue; // 任务队列
    ngx_int_t waiting;             // 当前任务数量，值并非精准，且可能为负数
    ngx_thread_cond_t cond;        // 队列非空条件变量
    ngx_log_t *log;                // 日志
    ngx_str_t name;                // 线程池名称
    ngx_uint_t threads;            // 线程数量
    ngx_int_t max_queue;           // 最大任务数量

    // 对应配置项的文件和行号
    u_char *file;
    ngx_uint_t line;
};
```
