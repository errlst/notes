## event

`event` 为单个事件的结构。事件通常代表某种具体条件，如：套接字可读可写，或发出信号。但其也可以不代表任何条件，如：线程间通讯。

#### 创建

`event_new(ev_base, fd, event, cb, arg)`，创建 `event` 对象：

* _fd_，文件描述符、信号或-1。
* _event_：
  * `EV_READ`、`EV_WRITE`，文件描述符的读写事件。
  * `EV_SIGNAL`，信号事件，此时 _fd_ 为信号号码。
  * `EV_PERSIST`，让事件持久化。
  * `EV_ET`，使用边缘触发文件描述符事件。
* _cb_，原型为 `void(evutil_socket_t fd, short event, void *arg)` 的回调函数。

`event_free(ev)`，释放通过 `event_new()` 创建的对象。

`event_assign(ev, ev_base, fd, event, cb, arg)`，赋值已存在的 `event` 对象。

#### 挂起

`event_add(ev, timeout)`，挂起 `event` 对象，并设置超时时间。

* 事件的条件触发、超时、使用 `event_active()` 激活事件后，事件将处于活动状态，在 `event_base` 的事件循环中，将完成该事件的回调。回调完成后，如果为非持久事件，将其置于非挂起态。
* 超时后，回调函数接收到的事件为 `EV_TIMEOUT`。

`event_del(ev)`，将事件置于非挂起。

## event_base

`event_base` 是事件循环的核心结构，其是不透明结构（只能通过函数对其访问）。

#### 创建

`event_base_new()`，创建默认配置的 `event_base` 对象。

`event_base_new_with_config(config)`，创建具有指定配置属性的 `event_base` 对象。

`event_base_free(eb)`，释放 `event_base` 对象。

#### 循环

`event_base_dispatch(eb)`，开启事件循环，直到不再有待处理或活动的事件，或者被显示终止。

`event_base_loop(eb, flags)`，dispatch 的更灵活版本：

* `EVLOOP_ONCE`，只处理一个事件然后返回。
* `EVLOOP_NONBLOCK`，非阻塞的方式开启事件循环，即如果没有待处理事件，立即返回。
* `EVLOOP_NO_EXIT_ON_EMPTY`，持续事件循环直到显示终止。

> 这三个标志可以任意组合，如 `EVLOOP_NONBLOCK | EVLOOP_NO_EXIT_ON_EMPTY` 异步开启事件循环。



