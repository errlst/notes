`ngx_cycle_t` 对象存储从特定配置创建的 nginx 上下文，当前 cycle 通过全局变量 `ngx_cycle` 引用，并在 nginx 的 worker 进程启动时继承。

每次重新加载 nginx 配置时，都会从新的 nginx 配置创建对应的 cycle，旧的 cycle 会在新的 cycle 创建后删除。

```c
struct ngx_cycle_s {
    // 模块配置文件数组，使用四级指针的作用是方便解引用
    // 获取核心模块上下文的表达式为：
    // (ngx_core_conf_t *)ngx_get_conf(conf_ctx, module)
    void ****conf_ctx;

    ngx_pool_t *pool;

    // 配置未解析完成前，也需要进行日志操作
    // 此时 log 指向 old_cycle 的 new_log
    // 配置解析完成后，log 指向 new_log
    ngx_log_t *log;
    ngx_log_t new_log;

    ngx_uint_t log_use_stderr; /* unsigned  log_use_stderr:1; */

    // 连接的文件
    ngx_connection_t **files;

    // 空闲连接
    ngx_connection_t *free_connections;
    ngx_uint_t free_connection_n;

    // 注册的模块
    ngx_module_t **modules;
    ngx_uint_t modules_n;
    ngx_uint_t modules_used; /* unsigned  modules_used:1; */

    // 可复用连接
    ngx_queue_t reusable_connections_queue;
    ngx_uint_t reusable_connections_n;
    time_t connections_reuse_time;  // 上次复用的时间，可以防止短时间多次复用产生大量日志

    ngx_array_t listening; // ngx_listen_t 数组
    ngx_array_t paths; // nginx 需要用到的所有路径

    // 转储文件
    // 可以将请求、响应、缓存等数据保存到文件中以分析
    ngx_array_t config_dump;
    ngx_rbtree_t config_dump_rbtree;
    ngx_rbtree_node_t config_dump_sentinel;

    // 打开的文件
    ngx_list_t open_files;
    ngx_list_t shared_memory;

    // 连接、文件数量
    ngx_uint_t connection_n;
    ngx_uint_t files_n;

    // 连接、读写事件
    ngx_connection_t *connections;
    ngx_event_t *read_events;
    ngx_event_t *write_events;

    ngx_cycle_t *old_cycle;

    // 配置文件、参数、前缀
    ngx_str_t conf_file;
    ngx_str_t conf_param;
    ngx_str_t conf_prefix;

    ngx_str_t prefix; // 工作目录
    ngx_str_t error_log;
    ngx_str_t lock_file; // 锁文件
    ngx_str_t hostname;  // 主机名
};
```

## 创建

`ngx_init_cycle(old)`，创建 cycle 对象，并从 old 中继承尽可能多的资源。在 nginx 启动时，会预先创建一个 `init_cycle`，然后再通过 `ngx_init_cycle()` 初始化实际的 cycle。
