- [更新时间](#更新时间)
- [分配内存池](#分配内存池)
- [拷贝配置文件路径、工作路径和日志文件路径](#拷贝配置文件路径工作路径和日志文件路径)
- [拷贝配置文件长度和配置参数](#拷贝配置文件长度和配置参数)
- [初始化路径信息](#初始化路径信息)
- [初始化转储文件](#初始化转储文件)
- [初始化已打开文件数量](#初始化已打开文件数量)
- [初始化共享内存链表](#初始化共享内存链表)
- [初始化监听数组](#初始化监听数组)
- [初始化模块配置数组](#初始化模块配置数组)
- [初始化主机名](#初始化主机名)
- [拷贝模块](#拷贝模块)

`nginx_init_cycle(old_cycle)` 的初始化流程为：

## 更新时间

```c
// 通过平台特定函数，强制更新时区
ngx_timezone_update();

// 重置 ngx_cached_time
tp = ngx_timeofday();
tp->sec = 0;

ngx_time_update();

log = old_cycle->log;
```

## 分配内存池

```c
pool = ngx_create_pool(NGX_CYCLE_POOL_SIZE, log);
if (pool == NULL) {
    return NULL;
}
pool->log = log;

cycle = ngx_pcalloc(pool, sizeof(ngx_cycle_t));
if (cycle == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}

cycle->pool = pool;
cycle->log = log;
cycle->old_cycle = old_cycle;
```

## 拷贝配置文件路径、工作路径和日志文件路径

```c
// 配置文件路径
cycle->conf_prefix.len = old_cycle->conf_prefix.len;
cycle->conf_prefix.data = ngx_pstrdup(pool, &old_cycle->conf_prefix);
if (cycle->conf_prefix.data == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}

// 工作路径
cycle->prefix.len = old_cycle->prefix.len;
cycle->prefix.data = ngx_pstrdup(pool, &old_cycle->prefix);
if (cycle->prefix.data == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}

// 日志文件
cycle->error_log.len = old_cycle->error_log.len;
cycle->error_log.data = ngx_pnalloc(pool, old_cycle->error_log.len + 1);
if (cycle->error_log.data == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}
ngx_cpystrn(cycle->error_log.data, old_cycle->error_log.data, old_cycle->error_log.len + 1);
```

> `conf_prefix` 和 `prefix` 作为前缀，因此不会放在字符串末尾，所以其不以空字符结尾。
>
> `error_log` 作为文件名，作为后缀，因此需要以空字符结尾，`ngx_cpystrn` 会在字符串后面加上空字符。

## 拷贝配置文件长度和配置参数

```c
cycle->conf_file.len = old_cycle->conf_file.len;
cycle->conf_file.data = ngx_pnalloc(pool, old_cycle->conf_file.len + 1);
if (cycle->conf_file.data == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}
ngx_cpystrn(cycle->conf_file.data, old_cycle->conf_file.data, old_cycle->conf_file.len + 1);

cycle->conf_param.len = old_cycle->conf_param.len;
cycle->conf_param.data = ngx_pstrdup(pool, &old_cycle->conf_param);
if (cycle->conf_param.data == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}
```

## 初始化路径信息

```c
n = old_cycle->paths.nelts ? old_cycle->paths.nelts : 10;
if (ngx_array_init(&cycle->paths, pool, n, sizeof(ngx_path_t *)) != NGX_OK) {
    ngx_destroy_pool(pool);
    return NULL;
}
ngx_memzero(cycle->paths.elts, n * sizeof(ngx_path_t *));
```

## 初始化转储文件

```c
if (ngx_array_init(&cycle->config_dump, pool, 1, sizeof(ngx_conf_dump_t)) != NGX_OK) {
    ngx_destroy_pool(pool);
    return NULL;
}

ngx_rbtree_init(&cycle->config_dump_rbtree, &cycle->config_dump_sentinel, ngx_str_rbtree_insert_value);
```

## 初始化已打开文件数量

```c
if (old_cycle->open_files.part.nelts) {
    n = old_cycle->open_files.part.nelts;
    for (part = old_cycle->open_files.part.next; part; part = part->next) {
        n += part->nelts;
    }
} else {
    n = 20;
}
```

## 初始化共享内存链表

```c
if (old_cycle->shared_memory.part.nelts) {
    n = old_cycle->shared_memory.part.nelts;
    for (part = old_cycle->shared_memory.part.next; part; part = part->next) {
        n += part->nelts;
    }
} else {
    n = 1;
}

if (ngx_list_init(&cycle->shared_memory, pool, n, sizeof(ngx_shm_zone_t)) != NGX_OK) {
    ngx_destroy_pool(pool);
    return NULL;
}
```

## 初始化监听数组

```c
n = old_cycle->listening.nelts ? old_cycle->listening.nelts : 10;
if (ngx_array_init(&cycle->listening, pool, n, sizeof(ngx_listening_t)) != NGX_OK) {
    ngx_destroy_pool(pool);
    return NULL;
}
ngx_memzero(cycle->listening.elts, n * sizeof(ngx_listening_t));

ngx_queue_init(&cycle->reusable_connections_queue);
```

## 初始化模块配置数组

```c
cycle->conf_ctx = ngx_pcalloc(pool, ngx_max_module * sizeof(void *));
if (cycle->conf_ctx == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}
```

## 初始化主机名

```c
if (gethostname(hostname, NGX_MAXHOSTNAMELEN) == -1) {
    ngx_log_error(NGX_LOG_EMERG, log, ngx_errno, "gethostname() failed");
    ngx_destroy_pool(pool);
    return NULL;
}

// 如果主机名过长，Linux 平台会截断且不保证以空字符结尾
hostname[NGX_MAXHOSTNAMELEN - 1] = '\0';
cycle->hostname.len = ngx_strlen(hostname);

cycle->hostname.data = ngx_pnalloc(pool, cycle->hostname.len);
if (cycle->hostname.data == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}

ngx_strlow(cycle->hostname.data, (u_char *)hostname, cycle->hostname.len);
```

## 拷贝模块

```c
// 拷贝 ngx_modules 静态模块
if (ngx_cycle_modules(cycle) != NGX_OK) {
    ngx_destroy_pool(pool);
    return NULL;
}

// 创建核心模块的配置
for (i = 0; cycle->modules[i]; i++) {
    if (cycle->modules[i]->type != NGX_CORE_MODULE) {
        continue;
    }

    module = cycle->modules[i]->ctx;

    if (module->create_conf) {
        rv = module->create_conf(cycle);
        if (rv == NULL) {
            ngx_destroy_pool(pool);
            return NULL;
        }
        cycle->conf_ctx[cycle->modules[i]->index] = rv;
    }
}

// environ 是外部全局变量，存放系统全局变量信息
senv = environ;
```
