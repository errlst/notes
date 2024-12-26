```c
/// os/unix/ngx_process_cycle.h

#define NGX_PROCESS_SINGLE     0
#define NGX_PROCESS_MASTER     1
#define NGX_PROCESS_SIGNALLER  2
#define NGX_PROCESS_WORKER     3
#define NGX_PROCESS_HELPER     4
```

# NGX_PROCESS_WORKER

nginx 的所有工作进程存放在全局变量：

```c
ngx_int_t ngx_last_process; // worker 进程数量
ngx_process_t ngx_processes[NGX_MAX_PROCESSES];
```

工作进程的主要作用就是处理客户端连接，其工作函数是 `ngx_worker_process_cycle()`。

# NGX_PROCESS_SIGNALLER

信号进程，信号进程的作用就是向 nginx 主进程发起信号，通过 `nginx -s <signal>` 启动。

信号进程的处理程序为 `ngx_signal_process()`。

信号进程通过文件 `logs/nginx.pid` 找到主进程的 pid，然后发送信号。

# NGX_PROCESS_MASTER

主进程。工作函数是 `ngx_master_process_cycle()`。

# NGX_PROCESS_SINGLE

单进程模式，此时由单个进程完成主进程和工作进程的任务。工作函数是 `ngx_single_process_cycle()`。

# NGX_PROCESS_HELPER

辅助进程，负责进行高速缓存管理和高速缓存加载，只有 unix 平台存在此类型的进程，运行时会创建辅助进程。工作函数是 `ngx_cache_manager_process_cycle()`。
