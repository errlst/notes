nginx 的大部分功能都是由模块实现的。

nginx 构建时，会生成文件 `ngx_modules.c`，其中包含 nginx 会用到的静态模块和动态模块的信息。

```c
/// ngx_modules.c

// 所有静态模块
ngx_module_t *ngx_modules[] = {...};
```

## ngx_module_t

```c
struct ngx_module_s {
    // 模块的版本、签名等信息
    ngx_uint_t ctx_index;
    ngx_uint_t index;
    char *name;
    ngx_uint_t spare0;
    ngx_uint_t spare1;
    ngx_uint_t version;
    const char *signature;

    void *ctx;               // 模块私有数据
    ngx_command_t *commands; // 模块支持的指令集
    ngx_uint_t type;         // 模块类型

    ngx_int_t (*init_master)(ngx_log_t *log);      // 无用
    ngx_int_t (*init_module)(ngx_cycle_t *cycle);  // 配置解析完成后，主进程中调用
    ngx_int_t (*init_process)(ngx_cycle_t *cycle); // 工作进程创建后调用
    ngx_int_t (*init_thread)(ngx_cycle_t *cycle);  // 无用
    void (*exit_thread)(ngx_cycle_t *cycle);       // 无用
    void (*exit_process)(ngx_cycle_t *cycle);      // 工作进程退出时调用
    void (*exit_master)(ngx_cycle_t *cycle);       // 主进程退出时调用

    // 预留拓展
    uintptr_t spare_hook0;
    uintptr_t spare_hook1;
    uintptr_t spare_hook2;
    uintptr_t spare_hook3;
    uintptr_t spare_hook4;
    uintptr_t spare_hook5;
    uintptr_t spare_hook6;
    uintptr_t spare_hook7;
};
```

模块类型定义 `ctx` 具体存储什么数据，nginx 内置以下几种类型模块：

```c
#define NGX_CORE_MODULE     0x45524F43  /* "CORE" */
#define NGX_CONF_MODULE     0x464E4F43  /* "CONF" */
#define NGX_EVENT_MODULE    0x544E5645  /* "EVNT" */
#define NGX_HTTP_MODULE     0x50545448  /* "HTTP" */
#define NGX_MAIL_MODULE     0x4C49414D  /* "MAIL" */
#define NGX_STREAM_MODULE   0x4d525453  /* "STRM" */
```

### 核心模块

核心模块包括 `ngx_core_module`、`ngx_errlog_module`、`ngx_events_module`、`ngx_http_module` 等。

核心模块的上下文定义为：

```c
typedef struct {
    ngx_str_t name;                                     // 模块字符串
    void *(*create_conf)(ngx_cycle_t *cycle);           // 解析配置前调用，返回初始化完成后的配置上下文
    char *(*init_conf)(ngx_cycle_t *cycle, void *conf); // 所有配置解析完成后调用，conf 为解析后的配置上下文
} ngx_core_module_t;
```