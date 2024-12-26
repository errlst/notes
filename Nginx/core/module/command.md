`ngx_command_t` 定义了一个配置指令，每个支持配置的模块都包含一个这样的结构数组，描述如何处理模块的配置。nginx 预定义了 `ngx_null_command` 作为数组的结束哨兵。

```c
struct ngx_command_s {
    ngx_str_t name;                                               // 指令名
    ngx_uint_t type;                                              // 指令类型
    char *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf); // 处理转换函数
    ngx_uint_t conf;                                              // 对应的配置上下文
    ngx_uint_t offset;                                            // 配置上下文中对应字段的偏移
    void *post;
};
```

# type

`type` 是一个标志位字段，指定参数以下属性。

- 参数数量。

  - `NGX_CONF_NOARGS`，指令无需参数。

  - `NGX_CONF_TAKE1 ~ NGX_CONF_TAKE7`，接受指定数量参数。可以通过或运算，表示指令可以采用不同数量的参数，但必须使用指定数量的参数，如 `NGX_CONF_TAKE1 | NGX_CONF_TAKE2` 表示接受 1 个或 2 个参数。

  - `NGX_CONF_1MORE ~ NGX_CONF_2MORE` 至少接受指定数量参数。

- 指令类型。

  - `NGX_CONF_BLOCK`，块指令，可以在 `{}` 中包含其他指令，甚至可以提供解析器独立处理内部内容。

  - `NGX_CONF_FLAG`，布尔指令，值要么是 `on` 要么是 `off`。

- 指令可以出现的上下文。

  - `NGX_MAIN_CONF`，顶层上下文，用于整个 nginx 实例。

  - `NGX_DIRECT_CONF`，配置只在当前配置块中有效，不会传递给子配置或受父配置影响。如 http 模块的 `log_format`、http.server 的 `proxy_pass`。

  - `NGX_EVENT_CONF`，events 块。

  - `NGX_HTTP_MAIN_CONF`，http 块。

  - `NGX_HTTP_LOC_CONF`，http 的 location 块。

# set

处理转换函数处理指令，并将解析后的值存储到对应的配置中。

nginx 内置了一些常用的处理程序：

- `ngx_conf_set_flag_slot()`，将 on 和 off 转到 0 和 1。

- `ngx_conf_set_str_slot()`，存储为 `ngx_str_t`。

- `ngx_conf_set_str_array_slot()`，存储为 `ngx_array_t`，元素为 `ngx_str_t`。

- `ngx_conf_set_keyval_slot()`，存储为 `ngx_array_t`，元素为 `ngx_keyval_t`。

- `ngx_conf_set_num_slot()`，存储为 `ngx_int_t`。

- `ngx_conf_set_size_slot()`，将 size 格式存储为 `size_t`。

  > size 格式是以 k、K、m、M 作为后缀的数字。

- `ngx_conf_set_sec_slot()`，将 time 格式存储为 `time_t`，单位为毫秒。

> time 格式是以 y、M、w、d、h、m、s、ms 后缀表示的时间，每年固定 365 天，每月固定 30 天。
>
> 如 `1h30m`，`90m`、`5400s` 表示相同时间。

- `ngx_conf_set_bufs_slot()`，将两个参数转换为 `ngx_bufs_t`。

# flag

`flag` 指定指令可以设置的配置上下文。

核心模块忽略该属性，核心模块的配置上下文只有 `NGX_MAIN_CONF` 和 `NGX_DIRECT_CONF` 两种指令可访问。
i