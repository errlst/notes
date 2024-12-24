## 日志类型

nginx 支持以下几种类型的日志输出：

- 输出到 stderr。

- 输出到文件。

- 输出到 syslog。

- 输出到内存。

## ngx_log_t

`ngx_log_t` 是链式结构，当记录日志时，会将消息传递到链上的所有 logger。

```c
typedef u_char *(*ngx_log_handler_pt)(ngx_log_t *log, u_char *buf, size_t len);
typedef void (*ngx_log_writer_pt)(ngx_log_t *log, ngx_uint_t level, u_char *buf, size_t len);

struct ngx_log_s {
    ngx_uint_t log_level;
    ngx_open_file_t *file;

    ngx_atomic_uint_t connection;

    time_t disk_full_time;

    ngx_log_handler_pt handler;
    void *data;

    ngx_log_writer_pt writer;
    void *wdata;

    /*
     * we declare "action" as "char *" because the actions are usually
     * the static strings and in the "u_char *" case we have to override
     * their types all the time
     */

    char *action;

    ngx_log_t *next;
};
```

## 初始化

`ngx_log_init(prefix, error_log)`，初始化全局定义的 logger 并返回。

- `prefix`，日志文件路径。

- `error_log`，错误日志文件，如果为空，则输出到 stderr。

> `ngx_log_init` 的主要作用是确认日志文件名和路径，然后打开文件。

```c
void test_log() {
    ngx_log_t *log_1 = ngx_log_init((u_char *)"", (u_char *)"");
    ngx_log_t *log_2 = ngx_log_init((u_char *)"", (u_char *)"log");
    printf("%d\n", log_1 == log_2); // 1
}
```

## 日志输出

nginx 拥有两种不同的日志输出级别。

### 错误日志

错误日志根据日志等级判断是否输出到日志中。

> ```c
> /// ngx_log.h
> #define NGX_LOG_STDERR 0
> #define NGX_LOG_EMERG 1
> #define NGX_LOG_ALERT 2
> #define NGX_LOG_CRIT 3
> #define NGX_LOG_ERR 4
> #define NGX_LOG_WARN 5
> #define NGX_LOG_NOTICE 6
> #define NGX_LOG_INFO 7
> #define NGX_LOG_DEBUG 8
> ```

使用 `ngx_log_error()` 输出错误日志。

```c
#define ngx_log_error(level, log, ...)                \
    if ((log)->log_level >= level)                    \
    ngx_log_error_core(level, log, __VA_ARGS__)

void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log, ngx_err_t err, const char *fmt, ...);
```

> `ngx_log_error_core` 的主要作用就是构造日志消息，然后输出到日志链。

- `err`，错误代码，如果不为 0，会生成对应的提示信息加入日志。

#### 示例

```c
void test_log() {
    ngx_log_t *log = ngx_log_init((u_char *)"./", (u_char *)"log");
    log->log_level = NGX_LOG_ERR;
    for (ngx_uint_t level = NGX_LOG_DEBUG; level <= NGX_LOG_DEBUG; --level) {
        ngx_log_error(level, log, 0, "level = %d", level);
    }
}
```

```shell
nginx: [error] level = 4
nginx: [crit] level = 3
nginx: [alert] level = 2
nginx: [emerg] level = 1
nginx: [] level = 0
```

```text
2024/12/23 16:06:10 [error] 0#0: level = 4
2024/12/23 16:06:10 [crit] 0#0: level = 3
2024/12/23 16:06:10 [alert] 0#0: level = 2
2024/12/23 16:06:10 [emerg] 0#0: level = 1
2024/12/23 16:06:10 [] 0#0: level = 0
```

> 0#0 是 pid#tid

### debug 日志

debug 日志是特殊的错误日志，其日志等级依然是 `NGX_LOG_DEBUG`，但通过另一种掩码的方式决定是否输出到日志。

```c
#define NGX_LOG_DEBUG_CORE 0x010
#define NGX_LOG_DEBUG_ALLOC 0x020
#define NGX_LOG_DEBUG_MUTEX 0x040
#define NGX_LOG_DEBUG_EVENT 0x080
#define NGX_LOG_DEBUG_HTTP 0x100
#define NGX_LOG_DEBUG_MAIL 0x200
#define NGX_LOG_DEBUG_STREAM 0x400
```

使用 `ngx_log_debug()` 输出 debug 日志。

```c
#define ngx_log_debug(level, log, ...)                        \
    if ((log)->log_level & level)                             \
    ngx_log_error_core(NGX_LOG_DEBUG, log, __VA_ARGS__)
```
