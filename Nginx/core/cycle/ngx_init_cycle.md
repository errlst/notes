`nginx_init_cycle(old_cycle)` 的初始化流程为：

## 更新时间

```c
// 通过平台特定函数，强制更新时区
ngx_timezone_update();

tp = ngx_timeofday();
tp->sec = 0;
ngx_time_update();

log = old_cycle->log;
```

> 当 `tp->sec` 和 `current->sec` 相同时，`ngx_time_update()` 并不会对所有缓存的内容进行更新，因此将 `tp->sec` 设置为 0，确保会进行完整的缓存更新。

## 资源拷贝和内存预分配

```c
// 内存池
pool = ngx_create_pool(NGX_CYCLE_POOL_SIZE, log);
if (pool == NULL) {
    return NULL;
}
pool->log = log;

// new_cycle
cycle = ngx_pcalloc(pool, sizeof(ngx_cycle_t));
if (cycle == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}
cycle->pool = pool;
cycle->log = log;
cycle->old_cycle = old_cycle;

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

// 错误日志文件路径
// error_log 会作为 c 字符串处理，以空字符结尾
cycle->error_log.len = old_cycle->error_log.len;
cycle->error_log.data = ngx_pnalloc(pool, old_cycle->error_log.len + 1);
if (cycle->error_log.data == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}
ngx_cpystrn(cycle->error_log.data, old_cycle->error_log.data, old_cycle->error_log.len + 1);

// 配置文件路径
// 同 error_log
cycle->conf_file.len = old_cycle->conf_file.len;
cycle->conf_file.data = ngx_pnalloc(pool, old_cycle->conf_file.len + 1);
if (cycle->conf_file.data == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}
ngx_cpystrn(cycle->conf_file.data, old_cycle->conf_file.data, old_cycle->conf_file.len + 1);

// nginx 启动时通过 -g 传递的参数
cycle->conf_param.len = old_cycle->conf_param.len;
cycle->conf_param.data = ngx_pstrdup(pool, &old_cycle->conf_param);
if (cycle->conf_param.data == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}

// 路径
n = old_cycle->paths.nelts ? old_cycle->paths.nelts : 10;
if (ngx_array_init(&cycle->paths, pool, n, sizeof(ngx_path_t *)) != NGX_OK) {
    ngx_destroy_pool(pool);
    return NULL;
}
ngx_memzero(cycle->paths.elts, n * sizeof(ngx_path_t *));

// 转储文件
if (ngx_array_init(&cycle->config_dump, pool, 1, sizeof(ngx_conf_dump_t)) != NGX_OK) {
    ngx_destroy_pool(pool);
    return NULL;
}
ngx_rbtree_init(&cycle->config_dump_rbtree, &cycle->config_dump_sentinel, ngx_str_rbtree_insert_value);

// 预分配打开文件需要的内存
if (old_cycle->open_files.part.nelts) {
    n = old_cycle->open_files.part.nelts;
    for (part = old_cycle->open_files.part.next; part; part = part->next) {
        n += part->nelts;
    }
} else {
    n = 20;
}
if (ngx_list_init(&cycle->open_files, pool, n, sizeof(ngx_open_file_t)) != NGX_OK) {
    ngx_destroy_pool(pool);
    return NULL;
}

// 共享内存初始化
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

// 可复用链接队列
ngx_queue_init(&cycle->reusable_connections_queue);
```

## 模块相关初始化

```c
// 预分配 conf_ctx 内存
cycle->conf_ctx = ngx_pcalloc(pool, ngx_max_module * sizeof(void *));
if (cycle->conf_ctx == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}

// 主机名
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

// 拷贝 ngx_modules 静态模块
if (ngx_cycle_modules(cycle) != NGX_OK) {
    ngx_destroy_pool(pool);
    return NULL;
}
// 创建核心模块上下文
for (i = 0; cycle->modules[i]; i++) {
    if (cycle->modules[i]->type != NGX_CORE_MODULE) {
        continue;
    }

    module = cycle->modules[i]->ctx;

    if (module->create_conf) {
        rv = module->create_conf(cycle); // 创建核心模块的 ngx_core_conf_t 上下文
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

## 配置文件初始化

```c
// conf 定义为 ngx_conf_t，用于解析配置文件临时时使用
ngx_memzero(&conf, sizeof(ngx_conf_t));
conf.args = ngx_array_create(pool, 10, sizeof(ngx_str_t));
if (conf.args == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}
conf.temp_pool = ngx_create_pool(NGX_CYCLE_POOL_SIZE, log);
if (conf.temp_pool == NULL) {
    ngx_destroy_pool(pool);
    return NULL;
}
conf.ctx = cycle->conf_ctx;
conf.cycle = cycle;
conf.pool = pool;
conf.log = log;
conf.module_type = NGX_CORE_MODULE;
conf.cmd_type = NGX_MAIN_CONF;

// 解析 nginx 启动时 -g 传递的参数
if (ngx_conf_param(&conf) != NGX_CONF_OK) {
    environ = senv;
    ngx_destroy_cycle_pools(&conf);
    return NULL;
}

// 解析配置文件
if (ngx_conf_parse(&conf, &cycle->conf_file) != NGX_CONF_OK) {
    environ = senv;
    ngx_destroy_cycle_pools(&conf);
    return NULL;
}

// ngx_test_config，只检查配置文件是否有问题，对应 -t、-T 参数
// ngx_quiet_mode，抑制错误输出，对应 -q 参数
if (ngx_test_config && !ngx_quiet_mode) {
    ngx_log_stderr(0, "the configuration file %s syntax is ok", cycle->conf_file.data);
}

// 初始化核心模块的配置
for (i = 0; cycle->modules[i]; i++) {
    if (cycle->modules[i]->type != NGX_CORE_MODULE) {
        continue;
    }

    module = cycle->modules[i]->ctx;

    if (module->init_conf) {
        if (module->init_conf(cycle, cycle->conf_ctx[cycle->modules[i]->index]) == NGX_CONF_ERROR) {
            environ = senv;
            ngx_destroy_cycle_pools(&conf);
            return NULL;
        }
    }
}

// 如果是信号进程，此时就可以结束了
if (ngx_process == NGX_PROCESS_SIGNALLER) {
    return cycle;
}
```

## 相关文件和路径创建

```c
// 创建 pid 文件
ccf = (ngx_core_conf_t *)ngx_get_conf(cycle->conf_ctx, ngx_core_module);
if (ngx_test_config) {
    if (ngx_create_pidfile(&ccf->pid, log) != NGX_OK) {
        goto failed;
    }
} else if (!ngx_is_init_cycle(old_cycle)) {
    old_ccf = (ngx_core_conf_t *)ngx_get_conf(old_cycle->conf_ctx, ngx_core_module);
    if (ccf->pid.len != old_ccf->pid.len || ngx_strcmp(ccf->pid.data, old_ccf->pid.data) != 0) {
        if (ngx_create_pidfile(&ccf->pid, log) != NGX_OK) {
            goto failed;
        }
        ngx_delete_pidfile(old_cycle);
    }
}

// 初始化锁文件
// 大多数系统上，锁机制通过原子操作实现，因此会忽略配置文件中的 lock_file 指令
// 对于其他系统，默认 lock 文件路径为 logs/nginx.lock
if (ngx_test_lockfile(cycle->lock_file.data, log) != NGX_OK) {
    goto failed;
}

// 创建临时需要的路径
if (ngx_create_paths(cycle, ccf->user) != NGX_OK) {
    goto failed;
}

// 打开默认日志文件
if (ngx_log_open_default(cycle) != NGX_OK) {
    goto failed;
}

// 打开所有需要的文件
part = &cycle->open_files.part;
file = part->elts;
for (i = 0; /* void */; i++) {
    if (i >= part->nelts) {
        if (part->next == NULL) {
            break;
        }
        part = part->next;
        file = part->elts;
        i = 0;
    }
    if (file[i].name.len == 0) {
        continue;
    }

    file[i].fd =
        ngx_open_file(file[i].name.data, NGX_FILE_APPEND, NGX_FILE_CREATE_OR_OPEN, NGX_FILE_DEFAULT_ACCESS);

    if (file[i].fd == NGX_INVALID_FILE) {
        ngx_log_error(NGX_LOG_EMERG, log, ngx_errno, ngx_open_file_n " \"%s\" failed", file[i].name.data);
        goto failed;
    }

// 子进程中自动关闭文件描述符
#if !(NGX_WIN32)
    if (fcntl(file[i].fd, F_SETFD, FD_CLOEXEC) == -1) {
        ngx_log_error(NGX_LOG_EMERG, log, ngx_errno, "fcntl(FD_CLOEXEC) \"%s\" failed", file[i].name.data);
        goto failed;
    }
#endif
}

// 日志文件创建完成后更新 log
cycle->log = &cycle->new_log;
pool->log = &cycle->new_log;
```

## 创建共享内存

## 处理监听套接字

```c
// 处理遗留监听套接字
if (old_cycle->listening.nelts) {
    ls = old_cycle->listening.elts;
    for (i = 0; i < old_cycle->listening.nelts; i++) {
        ls[i].remain = 0;
    }

    nls = cycle->listening.elts;
    for (n = 0; n < cycle->listening.nelts; n++) {
        for (i = 0; i < old_cycle->listening.nelts; i++) {
            if (ls[i].ignore) {
                continue;
            }

            if (ls[i].remain) {
                continue;
            }

            if (ls[i].type != nls[n].type) {
                continue;
            }

            if (ngx_cmp_sockaddr(nls[n].sockaddr, nls[n].socklen, ls[i].sockaddr, ls[i].socklen, 1) == NGX_OK) {
                nls[n].fd = ls[i].fd;
                nls[n].inherited = ls[i].inherited;
                nls[n].previous = &ls[i];
                ls[i].remain = 1;

                if (ls[i].backlog != nls[n].backlog) {
                    nls[n].listen = 1;
                }

#if (NGX_HAVE_DEFERRED_ACCEPT && defined SO_ACCEPTFILTER)
                nls[n].deferred_accept = ls[i].deferred_accept;

                if (ls[i].accept_filter && nls[n].accept_filter) {
                    if (ngx_strcmp(ls[i].accept_filter, nls[n].accept_filter) != 0) {
                        nls[n].delete_deferred = 1;
                        nls[n].add_deferred = 1;
                    }

                } else if (ls[i].accept_filter) {
                    nls[n].delete_deferred = 1;

                } else if (nls[n].accept_filter) {
                    nls[n].add_deferred = 1;
                }
#endif
#if (NGX_HAVE_DEFERRED_ACCEPT && defined TCP_DEFER_ACCEPT)
                if (ls[i].deferred_accept && !nls[n].deferred_accept) {
                    nls[n].delete_deferred = 1;

                } else if (ls[i].deferred_accept != nls[n].deferred_accept) {
                    nls[n].add_deferred = 1;
                }
#endif

#if (NGX_HAVE_REUSEPORT)
                if (nls[n].reuseport && !ls[i].reuseport) {
                    nls[n].add_reuseport = 1;
                }
#endif
                break;
            }
        }

        if (nls[n].fd == (ngx_socket_t)-1) {
            nls[n].open = 1;
#if (NGX_HAVE_DEFERRED_ACCEPT && defined SO_ACCEPTFILTER)
            if (nls[n].accept_filter) {
                nls[n].add_deferred = 1;
            }
#endif
#if (NGX_HAVE_DEFERRED_ACCEPT && defined TCP_DEFER_ACCEPT)
            if (nls[n].deferred_accept) {
                nls[n].add_deferred = 1;
            }
#endif
        }
    }

}
// 创建新的监听套接字
else {
    ls = cycle->listening.elts;
    for (i = 0; i < cycle->listening.nelts; i++) {
        ls[i].open = 1;
#if (NGX_HAVE_DEFERRED_ACCEPT && defined SO_ACCEPTFILTER)
        if (ls[i].accept_filter) {
            ls[i].add_deferred = 1;
        }
#endif
#if (NGX_HAVE_DEFERRED_ACCEPT && defined TCP_DEFER_ACCEPT)
        if (ls[i].deferred_accept) {
            ls[i].add_deferred = 1;
        }
#endif
    }
}

// 打开监听套接字
if (ngx_open_listening_sockets(cycle) != NGX_OK) {
    goto failed;
}

// 配置监听套接字
if (!ngx_test_config) {
    ngx_configure_listening_sockets(cycle);
}
```

> `NGX_HAVE_DEFERRED_ACCEPT`，允许延迟 accept，开启后，将 `accept()` 的处理推迟到三次握手完成后有收到数据时，这样一旦 accept() 就可以马上读取数据，而不必等待数据到达。
>
> `SO_ACCEPTFILTER`，FreeBSD、macOS 等平台提供的 socket 选项。
>
> `TCP_DEFER_ACCEPT`，Linux 平台提供的 socket 选项。

## 提交 cycle 配置

```c
// 将 stderr 重定向到 cycle->log
// linux 平台通过 dup2(fd, STDERR_FILENO) 实现
if (!ngx_use_stderr) {
    (void)ngx_log_redirect_stderr(cycle);
}

pool->log = cycle->log;

// 初始化模块，调用 module->init_module(cycle)
if (ngx_init_modules(cycle) != NGX_OK) {
    exit(1);
}
```

## 释放 old_cycle 中不能复用的共享内存

```c
opart = &old_cycle->shared_memory.part;
oshm_zone = opart->elts;

// i 遍历旧的共享内存区域
for (i = 0; /* void */; i++) {
    if (i >= opart->nelts) {
        if (opart->next == NULL) {
            goto old_shm_zone_done;
        }
        opart = opart->next;
        oshm_zone = opart->elts;
        i = 0;
    }

    part = &cycle->shared_memory.part;
    shm_zone = part->elts;

    // n 遍历新的共享内存区域
    for (n = 0; /* void */; n++) {
        if (n >= part->nelts) {
            if (part->next == NULL) {
                break;
            }
            part = part->next;
            shm_zone = part->elts;
            n = 0;
        }
        // 保留 name、tag 相同的且可复用的共享内存区域
        // 可以复用的共享内存在 cycle 创建共享内存区域时就已经复用
        // 此处只是释放 old_cycle 中不需要复用的共享内存区域
        if (oshm_zone[i].shm.name.len != shm_zone[n].shm.name.len) {
            continue;
        }
        if (ngx_strncmp(oshm_zone[i].shm.name.data, shm_zone[n].shm.name.data, oshm_zone[i].shm.name.len) != 0) {
            continue;
        }
        if (oshm_zone[i].tag == shm_zone[n].tag && oshm_zone[i].shm.size == shm_zone[n].shm.size &&
            !oshm_zone[i].noreuse) {
            goto live_shm_zone;
        }

        break;
    }

    ngx_shm_free(&oshm_zone[i].shm);

live_shm_zone:

    continue;
}

old_shm_zone_done:
```

## 关闭 old_cycle 中不能复用的监听套接字

```c
ls = old_cycle->listening.elts;
for (i = 0; i < old_cycle->listening.nelts; i++) {
    if (ls[i].remain || ls[i].fd == (ngx_socket_t)-1) {
        continue;
    }

    if (ngx_close_socket(ls[i].fd) == -1) {
        ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno, ngx_close_socket_n " listening socket on %V failed",
                        &ls[i].addr_text);
    }

// 删除 UNIX 套接字文件
#if (NGX_HAVE_UNIX_DOMAIN)
    if (ls[i].sockaddr->sa_family == AF_UNIX) {
        u_char *name;

        name = ls[i].addr_text.data + sizeof("unix:") - 1;

        ngx_log_error(NGX_LOG_WARN, cycle->log, 0, "deleting socket %s", name);

        if (ngx_delete_file(name) == NGX_FILE_ERROR) {
            ngx_log_error(NGX_LOG_EMERG, cycle->log, ngx_socket_errno, ngx_delete_file_n " %s failed", name);
        }
    }
#endif
}
```

## 关闭 old_cycle 中不必要的文件

```c
part = &old_cycle->open_files.part;
file = part->elts;

for (i = 0; /* void */; i++) {
    if (i >= part->nelts) {
        if (part->next == NULL) {
            break;
        }
        part = part->next;
        file = part->elts;
        i = 0;
    }

    if (file[i].fd == NGX_INVALID_FILE || file[i].fd == ngx_stderr) {
        continue;
    }

    if (ngx_close_file(file[i].fd) == NGX_FILE_ERROR) {
        ngx_log_error(NGX_LOG_EMERG, log, ngx_errno, ngx_close_file_n " \"%s\" failed", file[i].name.data);
    }
}

ngx_destroy_pool(conf.temp_pool);
```

## 收尾工作

```c
// master 进程，old_cycle 是 init_cycle
if (ngx_process == NGX_PROCESS_MASTER || ngx_is_init_cycle(old_cycle)) {
    ngx_destroy_pool(old_cycle->pool);
    cycle->old_cycle = NULL;
    return cycle;
}

// 创建临时内存池
if (ngx_temp_pool == NULL) {
    ngx_temp_pool = ngx_create_pool(128, cycle->log);
    if (ngx_temp_pool == NULL) {
        ngx_log_error(NGX_LOG_EMERG, cycle->log, 0, "could not create ngx_temp_pool");
        exit(1);
    }

    n = 10;

    if (ngx_array_init(&ngx_old_cycles, ngx_temp_pool, n, sizeof(ngx_cycle_t *)) != NGX_OK) {
        exit(1);
    }

    ngx_memzero(ngx_old_cycles.elts, n * sizeof(ngx_cycle_t *));

    ngx_cleaner_event.handler = ngx_clean_old_cycles;
    ngx_cleaner_event.log = cycle->log;
    ngx_cleaner_event.data = &dumb;
    dumb.fd = (ngx_socket_t)-1;
}
ngx_temp_pool->log = cycle->log;

// old_cycle 添加到 ngx_old_cycles 数组
old = ngx_array_push(&ngx_old_cycles);
if (old == NULL) {
    exit(1);
}
*old = old_cycle;

// 创建一个定时器，30s 后调用一次 ngx_clean_old_cycles()
// 在 ngx_clean_old_cycles() 中，会遍历 ngx_old_cycyle
// 如果某个 old_cycle 暂时还无法清理，会将 ngx_cleaner_event 再加入定时器中
if (!ngx_cleaner_event.timer_set) {
    ngx_add_timer(&ngx_cleaner_event, 30000);
    ngx_cleaner_event.timer_set = 1;
}

return cycle;
```

> 立即清理 old_cycle 可能会破坏系统运行，导致资源被误删、请求中断或服务不可用。且定时器机制使得清理工作更加安全、稳定且分散，不影响系统关键路径。
>
> 因此将 old_cycle 加入 ngx_old_cycles 数组，并创建定时器 30s 后清理一次 ngx_old_cycles。

## 失败后现场还原
