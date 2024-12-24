`ngx_connection_t` 是套接字描述符的包装器。

nginx 通过 `worker_connection` 指令限制每个 nginx 工作进程的连接数量，所有 `ngx_connection_t` 结构都是工作进程启动时预创建的，然后存放在 cycle 的 connections 字段中。

## 辅助函数

`ngx_get_connection(socket, log)`，从 `ngx_cycle` 中获取一个包裹 socket 的连接。

> 因为 worker 进程的连接数量是固定的，因此当 `ngx_cycle->free_connections` 不够时，会调用 `ngx_drain_connection()` 以释放部分可复用连接。

`ngx_reusable_connection(conn, reusable)`，开启或禁用连接复用。会将 `conn` 插入 `ngx_cycle->reusable_connections_queue` 队列，或从其中移除。

## ngx_connection_t

```c
struct ngx_connection_s {
    // 模块特定数据
    void *data;

    // IO 操作完成之后的回调
    ngx_event_t *read;
    ngx_event_t *write;

    ngx_socket_t fd; // 套接字描述符

    // IO 操作
    ngx_recv_pt recv;
    ngx_send_pt send;
    ngx_recv_chain_pt recv_chain;
    ngx_send_chain_pt send_chain;

    ngx_listening_t *listening; // 创建连接的监听对象
    off_t sent;                 // 连接总共发送的字节数
    ngx_log_t *log;
    ngx_pool_t *pool;

    int type; // socket类型
    struct sockaddr *sockaddr;
    socklen_t socklen;
    ngx_str_t addr_text; // 客户端地址的字符串表示

    ngx_proxy_protocol_t *proxy_protocol; // proxy 协议客户端地址

// QUIC 协议
#if (NGX_QUIC || NGX_COMPAT)
    ngx_quic_stream_t *quic;
#endif

// SSL 协议
#if (NGX_SSL || NGX_COMPAT)
    ngx_ssl_connection_t *ssl;
#endif

    ngx_udp_connection_t *udp; // udp 连接

    // 服务器地址
    struct sockaddr *local_sockaddr;
    socklen_t local_socklen;

    ngx_buf_t *buffer;        // 临时缓冲区，接受数据
    ngx_queue_t queue;        // 用于各种连接队列的管理（如空闲连接队列
    ngx_atomic_uint_t number; // 唯一连接标识符
    ngx_msec_t start_time;    // 连接开始事件
    ngx_uint_t requests;      // 连接处理的请求数

    unsigned buffered : 8;       // 缓冲区状态
    unsigned log_error : 3;      // 错误日志级别，ngx_connection_log_error_e 类型
    unsigned timedout : 1;       // 连接超时
    unsigned error : 1;          // 连接发生错误
    unsigned destroyed : 1;      // 连接被销毁
    unsigned pipeline : 1;       // 使用管道模式
    unsigned idle : 1;           // 连接空闲
    unsigned reusable : 1;       // 连接可复用
    unsigned close : 1;          // 连接需要被关闭
    unsigned shared : 1;         // 是否在进程、线程间共享
    unsigned sendfile : 1;       // 正在发送文件中的数据
    unsigned sndlowat : 1;       // 缓冲区数据高于低水位时才发送数据
    unsigned tcp_nodelay : 2;    // tcp nodelay 特性，ngx_connection_tcp_nodelay_e 类型
    unsigned tcp_nopush : 2;     // tcp nopush 特性，ngx_connection_tcp_nopush_e 类型
    unsigned need_last_buf : 1;  // 用于 http filter 模块
    unsigned need_flush_buf : 1; // 需要刷新缓冲区

#if (NGX_HAVE_SENDFILE_NODISKIO || NGX_COMPAT)
    unsigned busy_count : 2;
#endif

// 多线程文件发送
#if (NGX_THREADS || NGX_COMPAT)
    ngx_thread_task_t *sendfile_task;
#endif
};
```
