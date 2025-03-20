spdlog 提供了以下调优项：

- `SPDLOG_CLOCK_COARSE`，在 linux 平台下可以使用更快的 CLOCK_REALTIME_COARSE，但这个时钟可能误差会更大。

  <div style="display:flex;gap:10px;">

  ```cpp
  static auto bench_systcm_clock(benchmark::State &state) -> void {
    for (auto _ : state) {
      std::chrono::system_clock::now();
    }
  }
  BENCHMARK(bench_systcm_clock);

  static auto bench_steady_clock(benchmark::State &state) -> void {
    for (auto _ : state) {
      std::chrono::steady_clock::now();
    }
  }
  BENCHMARK(bench_steady_clock);

  static auto bench_realtime_clock(benchmark::State &state) -> void {
    for (auto _ : state) {
      auto ts = timespec{};
      clock_gettime(CLOCK_REALTIME_COARSE, &ts);
    }
  }
  BENCHMARK(bench_realtime_clock);

  BENCHMARK_MAIN();
  ```

  ```shell
  ---------------------------------------------------------------
  Benchmark                     Time             CPU   Iterations
  ---------------------------------------------------------------
  bench_systcm_clock         42.7 ns         39.5 ns     17788810
  bench_steady_clock         44.2 ns         40.8 ns     17949059
  bench_realtime_clock       13.2 ns         12.2 ns     54596648
  ```

  </div>

- `SPDLOG_NO_THREAD_ID`，开启后，日志调用时不会查询线程 id。

- `SPDLOG_NO_TLS`，开启后，spdlog 不会使用 thread_local。

- `SPDLOG_NO_ATOMIC_LEVELS`，开启后，日志等级使用非原子变量。

- `SPDLOG_USE_STD_FORMAT`，开启后，使用 std::format 替换 fmt。

- `SPDLOG_PREVENT_CHILD_FD`，开启后，子进程不会继承日志文件描述符。

- `SPDLOG_DISABLE_DEFAULT_LOGGER`，开启后，不会自动创建 default logger。

- `SPDLOG_NO_SOURCE_LOC`，开启后，`SPDLOG_LOGGER_DEBUG` 等宏函数不会记录 sourceloc 信息。

- `SPDLOG_ACTIVE_LEVEL`，编译期设置日志等级，影响 `SPDLOG_LOGGER_DEBUG` 等宏函数。
