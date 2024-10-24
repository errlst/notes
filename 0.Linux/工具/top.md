top 输出解析

```shell
top - 10:23:48 up 22:46,  1 user,  load average: 5.93, 3.32, 1.50
任务: 383 total,   1 running, 382 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  1.1 sy,  0.0 ni, 98.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7894.3 total,    999.0 free,   4332.8 used,   3069.7 buff/cache
MiB Swap:   4096.0 total,   4091.2 free,      4.8 used.   3561.5 avail Mem
```

#### 第一行：系统运行时间和负载

- 10:23:48： 当前时间

- up 22:46： 系统已运行 22 小时 46 分钟

- 1 user： 当前登陆的用户数量

- 5.93 3.32 1.50： 最近 1 分钟、5 分钟和 15 分钟的系统平均负载

#### 第二行：系统和进程状态

- 383 total：当前共有 383 个进程

- 1 running：当前有 1 个正在运行的进程

- 382 sleeping：当前有 382 个处于睡眠的进程

- 0 stopped：被停止的进程

- 0 zombie：僵尸进程

#### 第三行：cpu 使用情况

- 0.0 us：用户空间消耗 cpu 占比

- 1.1 sy：系统空间消耗 cpu 占比

- 0.0 ni：调整过优先级的进程 cpu 占比

- 98.9 id：cpu 空闲时间占比

- 0.0 wa：io 等待时间

- 0.0 hi：硬中断消耗 cpu 占比

- 0.0 si：软中断消耗 cpu 占比

#### 第四行：内存使用情况

#### 第五行：交换分区使用情况
