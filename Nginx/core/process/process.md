nginx 有以下几种类型的进程，当前进程的类型保存在 `ngx_process` 变量中：

- `NGX_PROCESS_SINGLE`，单进程模式，通过指令 `master_process off` 设置。单进程的循环函数为 `ngx_single_process_cycle()`。

- `NGX_PROCESS_MASTER`，主进程，主进程的循环函数为 `ngx_master_process_cycle()`。

> `ngx_master_process_cycle()` 中会创建固定数量的工作进程，工作进程数量通过指令 `worker_processes n` 设置（默认为 1），然后阻塞在 `sigsuspend()` 等待信号。

# 工作进程

nginx 的所有 worker 进程存放在全局变量：

```c
ngx_int_t ngx_last_process; // worker 进程数量
ngx_process_t ngx_processes[NGX_MAX_PROCESSES];
```

# 信号进程

`NGX_PROCESS_SIGNALLER`，信号进程，信号进程的作用就是向 nginx 主进程发起信号，通过 `nginx -s signal` 启动。

信号进程的处理程序为 `ngx_signal_process()`。

信号进程通过文件 `logs/nginx.pid` 找到主进程的 pid，然后发送信号。

## 信号类型

nginx 定义了一些平台无关的信号：

```c
#define NGX_SHUTDOWN_SIGNAL      QUIT
#define NGX_TERMINATE_SIGNAL     TERM
#define NGX_NOACCEPT_SIGNAL      WINCH
#define NGX_RECONFIGURE_SIGNAL   HUP

#if (NGX_LINUXTHREADS)
#define NGX_REOPEN_SIGNAL        INFO
#define NGX_CHANGEBIN_SIGNAL     XCPU

#else
#define NGX_REOPEN_SIGNAL        USR1
#define NGX_CHANGEBIN_SIGNAL     USR2
#endif
```

master 和 single 进程会处理以上所有信号，worker 和 helper 进程不会处理 `NGX_RECONFIGURE_SIGNAL` 和 `NGX_CHANGEBIN_SIGNAL`。

### NGX_SHUTDOWN_SIGNAL

正常关闭，全局变量 `ngx_quit` 设置为 1。

master 进程将信号转发到 worker 进程，然后关闭监听，等待所有 worker 进程关闭后退出进程。

`nginx -s quit` 发送信号。

### NGX_TERMINATE_SIGNAL
