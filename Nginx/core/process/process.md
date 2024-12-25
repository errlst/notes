nginx 有以下几种类型的进程，当前进程的类型保存在 `ngx_process` 变量中：

- `NGX_PROCESS_SINGLE`，单进程模式，通过指令 `master_process off` 设置。单进程的循环函数为 `ngx_single_process_cycle()`。

- `NGX_PROCESS_MASTER`，主进程，主进程的循环函数为 `ngx_master_process_cycle()`。

> `ngx_master_process_cycle()` 中会创建固定数量的工作进程，工作进程数量通过指令 `worker_processes n` 设置（默认为 1），然后阻塞在 `sigsuspend()` 等待信号。

# NGX_PROCESS_WORKER

nginx 的所有工作进程存放在全局变量：

```c
ngx_int_t ngx_last_process; // worker 进程数量
ngx_process_t ngx_processes[NGX_MAX_PROCESSES];
```

工作进程的主要作用就是处理客户端连接，其工作函数是 `ngx_worker_process_cycle()`。

# NGX_PROCESS_SIGNALLER

信号进程，信号进程的作用就是向 nginx 主进程发起信号，通过 `nginx -s signal` 启动。

信号进程的处理程序为 `ngx_signal_process()`。

信号进程通过文件 `logs/nginx.pid` 找到主进程的 pid，然后发送信号。

