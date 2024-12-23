`ngx_cycle_t` 对象存储从特定配置创建的 nginx 上下文，当前 cycle 通过全局变量 `ngx_cycle` 引用，并在 nginx 的 worker 进程启动时继承。

每次重新加载 nginx 配置时，都会从新的 nginx 配置创建对应的 cycle，旧的 cycle 会在新的 cycle 创建后删除。

## 创建

`ngx_init_cycle(old)`，创建 cycle 对象，并从 old 中继承尽可能多的资源。在 nginx 启动时，会预先创建一个 `init_cycle`，然后再通过 `ngx_init_cycle()` 初始化实际的 cycle。
