# CPU 缓存一致性

缓存一致性协议（MESI）中，缓存总是处于以下四种状态之一：

| 状态      | 说明                                                        |
| --------- | ----------------------------------------------------------- |
| Modified  | 该缓存行被修改过，且保证不会出现在其他 CPU 核心的缓存行中。 |
| Exclusive | 该缓存行被当前 CPU 核心独占，且与内存数据一致。             |
| Shared    | 该缓存行对应的内存和其他 CPU 核心共享。                     |
| Invalid   | 该缓存行无效。                                              |

当一段数据在多个核心中共享时，如果一个核心要更新数据，需要向总线发送 Invalidate 广播消息，等到其他核心将缓存行置于 Invalid 状态后才能修改缓存数据。为了提高性能，CPU 引入了 _Store Buffer_ 和 _Invalidate Queue_。

## Store Buffer

在 CPU 和缓存中引入 _Store Buffer_，写操作直接放入 buffer 中，然后立即返回执行其它操作，具体的同步和写入操作由 buffer 执行。

> 如果在 buffer 的数据写入缓存前需要读取，直接从 buffer 中读取。

## Invalidate Queue

由于 _Store Buffer_ 容量有限，如果 buffer 发送 Invalidate 消息后，其它核心无法及时处理消息，则 buffer 很快就会写满。因此在缓存和内存中引入 _Invalidate Queue_，当 CPU 核心收到 Invalidate 消息后，将其放入 queue 后直接确认消息。

CPU 核心发送 Invalidate 消息前，需要先查看自己的 _Invalidate Queue_，确认所有 Invalidate 消息处理完之后才能发送。

# 内存屏障

## Store-Store 屏障（写屏障）

引入 _Store Buffer_ 会导致写入延迟。对于下述代码（忽略重排和 _Invalidate Queue_）：

```c
int a = 0, b = 0;

/* thread 0 on cpu 0 */
void work_0() {
    a = 1;
    b = 1;
}

/* thread 1 on cpu 1 */
void work_1() {
    while(b == 0) continue;
    assert(a == 1);
}
```

如果此时 cpu0 和 cpu1 的对 a 的缓存行状态为 Shared，可能发生将 `a=1` 的写操作放入 _Store Buffer_，然后 b 直接更新缓存。cpu1 读取到 b 为 1，但 `a=1` 依然在 buffer 中，读取的 a 为 0。

## Load-Load 屏障（读屏障）

对于上述代码（忽略 _Store Buffer_），如果 cpu0 和 cpu1 对 a 的缓存行状态为 Shared，当 cpu1 读取 a 时，queue 中的 Invalidate 消息还未处理，那么 cpu1 就会认为 a 依然处于 Shared 状态，读取 a 为 0。

## Load-Store 屏障

Load-Store 屏障可能发生在任何带有乱序执行的处理器上。

```c
int a = 0, b = 0;

/* thread 0 on cpu 0 */
void work_0() {
    int tmp0 = a;
    b = 1;
}

/* thread 1 on cpu 1 */
void work_1() {
    int tmp1 = b;
    a = 1;
}
```

cpu 可能将指令 `b=1` 和 `a=1` 放在 `tmp = ` 指令的前面，然后 `tmp0` 和 `tmp1` 都为 1。

## Store-Load 屏障（全屏障）

# TSO

x86 架构 CPU 采用的是 Total Store Ordering 一致性模型：

- x86cpu 没有 _Invalidate Queue_，因此不需要 Load-Load 屏障。

- x86cpu 的 _Store Buffer_ 实现为严格队列，因此不需要 Store-Store 屏障。

- x86cpu 的处理器乱序不会导致 Load-Store 问题。

因此，在 x86 架构上只需要考虑 Store-Load 屏障问题。
