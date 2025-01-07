# 源码结构

- auto，自动生成脚本。

  > auto/configure 检查编译环境，生成 make 文件。

- src。

  - core，基本类型和函数，数组、字符串、函数池等。

  - event，事件核心。

    - modules，事件通知模块，epoll、poll、select 等。

  - http，核心 http 模块。

    - v2，http/2

    - v3，http/3

    - module，http 其它模块。

  - mail，邮箱模块。

  - os，平台特定代码。

  - stream，流模块。

## 时间

nginx 使用 `ngx_time_t` 结构表示秒、毫秒和 GMT 偏移：

```c
// src/core/ngx_times.h
typedef struct {
    time_t sec;
    ngx_uint_t msec;
    ngx_int_t gmtoff;
} ngx_time_t;
```

### 获取时间

宏 `ngx_time()` 和 `ngx_timeofday()` 获取缓存的时间，也可通过 `ngx_gettimeofday()` 显示获取时间。

nginx 默认在事件循环开始时会更新缓存时间，也可通过调用 `ngx_time_update()` 立即更新缓存时间。

#### 格式化字符串

nginx 定义了一些全局字符串，使用不同格式表示当前时间：

```c
// src/core/ngx_time.h
extern volatile ngx_str_t ngx_cached_err_log_time;
extern volatile ngx_str_t ngx_cached_http_time;
extern volatile ngx_str_t ngx_cached_http_log_time;
extern volatile ngx_str_t ngx_cached_http_log_iso8601;
extern volatile ngx_str_t ngx_cached_syslog_time;
```

- `ngx_cached_err_log_time`，用于错误日志。

- `ngx_cached_http_time`，用于 HTTP 头。

- `ngx_cached_http_log_time`，用于 HTTP 访问日志。

- `ngx_cached_http_log_iso8601`，ISO8601 标准格式。

- `ngx_cached_syslog_time`，用于 syslog。

# 错误处理

nginx 中大多数函数返回以下代码：

- `NGX_OK`，正常返回。

- `NGX_ERROR`，操作失败。

- `NGX_AGAIN`，操作未完成，需要再次调用。

- `NGX_BUSY`，资源不可用。

- `NGX_DONE`，操作完成或在其他地方继续。

- `NGX_DECLINED`，操作被拒绝，如在配置中禁用。

- `NGX_ABORT`，函数中止。
