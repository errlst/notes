全局变量 `signals` 存储所有 nginx 需要处理的信号。

```c
/// os/unix/ngx_process.c
ngx_signal_t signals[] = {
    {ngx_signal_value(NGX_RECONFIGURE_SIGNAL), "SIG" ngx_value(NGX_RECONFIGURE_SIGNAL), "reload", ngx_signal_handler},
    {ngx_signal_value(NGX_REOPEN_SIGNAL), "SIG" ngx_value(NGX_REOPEN_SIGNAL), "reopen", ngx_signal_handler},
    {ngx_signal_value(NGX_NOACCEPT_SIGNAL), "SIG" ngx_value(NGX_NOACCEPT_SIGNAL), "", ngx_signal_handler},
    {ngx_signal_value(NGX_TERMINATE_SIGNAL), "SIG" ngx_value(NGX_TERMINATE_SIGNAL), "stop", ngx_signal_handler},
    {ngx_signal_value(NGX_SHUTDOWN_SIGNAL), "SIG" ngx_value(NGX_SHUTDOWN_SIGNAL), "quit", ngx_signal_handler},
    {ngx_signal_value(NGX_CHANGEBIN_SIGNAL), "SIG" ngx_value(NGX_CHANGEBIN_SIGNAL), "", ngx_signal_handler},
    {SIGALRM, "SIGALRM", "", ngx_signal_handler},
    {SIGINT, "SIGINT", "", ngx_signal_handler},
    {SIGIO, "SIGIO", "", ngx_signal_handler},
    {SIGCHLD, "SIGCHLD", "", ngx_signal_handler},
    {SIGSYS, "SIGSYS, SIG_IGN", "", NULL},
    {SIGPIPE, "SIGPIPE, SIG_IGN", "", NULL},
    {0, NULL, "", NULL}};
```

# 平台无关信号

nginx 定义了以下平台无关信号，用于 nginx 特殊使用：

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

## NGX_SHUTDOWN_SIGNAL

正常关闭。

对应全局变量 `ngx_quit`。

主进程将信号转发到工作进程，然后关闭监听，等待所有工作进程关闭后退出进程。

可以通过 `nginx -s quit` 发送信号。

## NGX_TERMINATE_SIGNAL

对应全局变量 `ngx_terminate`。

可以通过 `nginx -s stop` 发送信号。

主进程会先将信号转发到工作进程，如果一段时间后工作进程依然没有退出，则直接向工作进程发送 `SIGKILL` 信号。

工作进程收到信号后，执行 `ngx_worker_process_exit()`。

## NGX_NOACCEPT_SIGNAL

让主进程不再接受新连接，同时让现有的连接正常运行直到断开。

对应全局变量 `ngx_noaccept`，且只有当主线程以守护进程运行时该信号有效。

主线程会向所有工作线程发送 `NGX_SHUTDOWN_SIGNAL` 信号。

## NGX_RECONFIGURE_SIGNAL

重新读取配置文件，并创建一个新的 cycle。

- 如果 new_cycle 创建失败，则不做处理。

- 否则，在 new_cycle 上创建新的工作进程，然后向之前的工作进程发送 `NGX_SHUTDOWN_SIGNAL` 信号。

对应变量 `ngx_reconfigure`。

## NGX_CHANGEBIN_SIGNAL

更新 nginx 二进制文件。

主进程重新启动 nginx 二进制文件，并传入所有监听套接字。`ngx_exec_new_binary()`。

对应变量 `ngx_change_binary`。

对于新的二进制文件，如果旧的二进制文件还在运行，则忽略该信号；对于旧的二进制文件，如果新的二进制文件已经运行，则忽略该信号。
