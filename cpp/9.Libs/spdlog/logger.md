## backtrace

backtrace 是一个环形队列，允许 backtrace 时，日志消息会同时向 backtrace 中写入一份。

<div style = "display:flex;gap:20px;">

```cpp
auto main() -> int {
  auto logger = spdlog::default_logger();
  logger->set_level(spdlog::level::debug);

  /* 开启 backtrace 并设置大小，且后面的日志会覆盖之前的日志 */
  logger->enable_backtrace(3);

  for (auto i = 0; i < 5; ++i) {
    logger->debug("{}", i);
  }

  /* 输出 backtrace */
  logger->dump_backtrace();

  return 0;
}
```

```shell
[2025-03-05 15:28:54.080] [debug] 0
[2025-03-05 15:28:54.080] [debug] 1
[2025-03-05 15:28:54.080] [debug] 2
[2025-03-05 15:28:54.080] [debug] 3
[2025-03-05 15:28:54.080] [debug] 4
[2025-03-05 15:28:54.080] [info] ****************** Backtrace Start ******************
[2025-03-05 15:28:54.080] [debug] 2
[2025-03-05 15:28:54.080] [debug] 3
[2025-03-05 15:28:54.080] [debug] 4
[2025-03-05 15:28:54.080] [info] ****************** Backtrace End ********************
```

</div>

## pattern

单个 pattern 的格式为 `%((-=)(number))(!)(flag)`。

- (-=)，对齐方式。

  - \-，右对齐。

  - =，居中对齐。

  - 默认，左对齐。

- (number)，显示宽度。

- (!)，是否截断超出宽度的部分。

- (flag)，spdlog 内置了以下 flag：

  <div style="display:flex;gap:20px;">
  <div>

  | 时间相关            | 说明                       | 示例                    |
  | ------------------- | -------------------------- | ----------------------- |
  | C <br> Y            | 年                         | 25 <br> 2025            |
  | b / h <br> B <br> m | 月                         | Mar <br> March <br> 03  |
  | d                   | 日                         |                         |
  | H <br> I            | 时(24 小时)<br>时(12 小时) | 16 <br> 04              |
  | M                   | 分                         |                         |
  | S                   | 秒                         |                         |
  | e                   | 毫秒                       |                         |
  | f                   | 微秒                       |                         |
  | F                   | 纳秒                       |                         |
  | a <br> A            | 星期                       | Wed <br> Wednesday      |
  | p                   | AM or PM                   |                         |
  | c                   | datetime                   | Wed Mar 5 16:24:37 2025 |
  | D / x               | date                       | 03/05/25                |
  | r <br> R            | clock                      | 04:24:37 PM <br> 16:24  |
  | T / X               | ISO 8601 format            | 16:24:37                |
  | z                   | timezone                   | +08:00                  |

  </div>
  <div style="display:flex;flex-direction:column;gap:20px;">
  <div>

  | 日志消息相关  | 说明                                 |
  | ------------- | ------------------------------------ |
  | v             | 消息文本                             |
  | ^ / $         | 颜色范围的起止                       |
  | @             | 日志消息的 source loc                |
  | s / g         | 日志消息的文件名                     |
  | #             | 日志消息文件行                       |
  | !             | 日志消息函数名                       |
  | %             | `%` 符号                             |
  | O / o / i / u | 自上个日志消息的秒、毫秒、微秒、纳秒 |

  </div>
  <div>

  | 其它  | 说明                  |
  | ----- | --------------------- |
  | +     | 默认格式              |
  | n     | 日志名称              |
  | l / L | 日志等级              |
  | t     | 创建日志消息的线程 id |
  | P     | pid                   |
  | &     | mdc                   |

  </div>
  </div>
  </div>

## mdc

mdc 是一个线程局部存储的键值对 map，mdc_formatter 会将 mdc 中的键值对以 `[k1:v1 k2:v2]` 的格式转换。

> 因为存放键值对的 map 是 `thread_local` 的，因此不能在异步日志中使用 mdc。

<div style="display:flex;gap:20px;">

```cpp
auto main() -> int {
  auto logger = spdlog::default_logger();
  spdlog::mdc::put("key1", "value1");
  spdlog::mdc::put("key2", "value2");
  logger->info("mdc test");
  return 0;
}
```

```shell
[2025-03-05 17:20:16.076] [info] [key1:value1 key2:value2] mdc test
```

</div>
