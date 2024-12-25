# 事件循环

`ngx_worker_process_cycle()` 是 worker 进程的事件循环，循环调用 `ngx_process_events_and_timers()` 处理 IO 和定时器事件。

# ngx_event_t

`ngx_event_t` 可以同时表示 IO 事件、定时器和发布事件。

```c
struct ngx_event_s {
    void *data; // 事件需要使用的上下文

    unsigned write : 1;           // IO事件标志，1 为写事件
    unsigned accept : 1;          // 新连接标志
    unsigned instance : 1;        // 探查 kqueue 和 epoll 中的陈旧事件
    unsigned active : 1;          // 事件已注册到内核；aio 模式下，标志操作已发布。
    unsigned disabled : 1;        // 事件被禁用
    unsigned ready : 1;           // 事件已就绪；aio 模式下，0 表示没有可发布的操作
    unsigned oneshot : 1;         // 一次性事件
    unsigned complete : 1;        // aio 操作完成
    unsigned eof : 1;             // 读到 eof
    unsigned error : 1;           // 发生错误
    unsigned timedout : 1;        // 事件超时
    unsigned timer_set : 1;       // 定时器是否被设置
    unsigned delayed : 1;         // 由于速率限制被延迟
    unsigned deferred_accept : 1; // 延迟接受的连接
    unsigned pending_eof : 1;     // socket 上的 eof，即使此时可能依然有数据可读
    unsigned posted : 1;          // 事件已加入循环队列
    unsigned closed : 1;          // 连接已关闭
    unsigned channel : 1;         // 测试 worker 进程退出时的通道事件
    unsigned resolver : 1;        // 解析器相关事件
    unsigned cancelable : 1;      // worker 进程关闭时忽略该事件，或者等待该事件完成后再关闭

// kqueue 相关
#if (NGX_HAVE_KQUEUE)
    unsigned kq_vnode : 1;
    int kq_errno;
#endif

    // kqueue 模型下，
    //  accept：    等待 accept 的 socket 数量
    //  read：      事件就绪时，需要读取的字节数
    //  write：     事件就绪时，缓冲区可用空间
    //
    // iocp 模型下，无意义
    //
    // other：
    //  accept：    1 表示等待 accept 的 socket 数量不止 1 个
    //  read：      事件就绪时，需要读取的字节数，-1 表示未知
    int available;

    ngx_event_handler_pt handler; // 事件回调

// iocp 相关
#if (NGX_HAVE_IOCP)
    ngx_event_ovlp_t ovlp;
#endif

    ngx_uint_t index; // 事件索引
    ngx_log_t *log;
    ngx_rbtree_node_t timer; // 定时器节点
    ngx_queue_t queue;       // 事件循环队列节点
};
```

## IO 事件

通过 `ngx_get_connection()` 获取的连接有 `read` 和 `write` 两个事件，用于接受 IO 变化的通知，且这些事件都在边缘触发模式下运行，无论底层 IO 机制如何。

## 定时器事件

`ngx_add_timer(event, msec)`，添加定时器事件。

> 如果事件已经被设置为定时器：
>
> - 如果新的到期时间点和旧的到期时间点差距小于 `NGX_TIMER_LAZY_DELAY`，则不做处理。
>
> - 否则，移除定时器。

`ngx_del_timer(event)`，移除定时器事件。

## 发布事件

发布事件的处理程序将在事件循环的某个节点被调用。

通常情况，通过表达式 `ngx_post_event(event, &ngx_posted_next_events)` 将事件加入到 `ngx_posted_event` 队列中，然后在事件循环的后期，通过 `ngx_event_process_posted()` 处理所有发布事件。

> 发布事件都是一次性事件。
>
> 在处理程序中不要将事件再次放入循环中，可能会导致程序一直循环调用处理程序。

`ngx_delete_posted_event(event)`，删除发布事件。
