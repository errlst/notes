#### async

`async(policy, f, args...)`，根据策略执行任务，并返回关联的 `future` 对象。

###### 启动策略
* `std::launch::async`，异步执行任务，如同在新线程中调用 `invoke(auto{forward<F>(f)}, auto{forward<Args>(args)}...)`。

  > mscv 实现使用内部线程池；gcc 和 clang 使用 `std::thread` 新建线程。

* `std::launch::delay`，延迟执行任务，对返回结果首次求值时，在求值线程中同步执行任务。

  ```cpp
  auto main() -> int {
      auto main_thread_id = std::this_thread::get_id();
      auto fut = std::async(std::launch::deferred, [&] {
          std::println("{}", main_thread_id == std::this_thread::get_id());
      });
      std::async(std::launch::async, [&] { fut.get(); }).get(); // false
      return 0;
  }
  ```

###### tips
如果 `async()` 返回值被忽略，`future` 析构时会阻塞直到任务完成。

#### future
`future<T>` 提供访问异步操作的结果或其抛出异常的机制。`future` 只对外提供了获取结果的接口，没有提供设置结果的接口。

###### 访问
`get()`，阻塞获取 `future` 存储的结果，或抛出异常。`future` 的结果只能获取一次，如果需要多次访问，使用 `shared_future`。

###### 析构
如果 `future` 由 `async` 返回，且析构时状态未就绪，将阻塞直到状态就绪。

#### promise

`promise<T>` 提供设置异步操作或抛出异常的机制，之后创建关联的 `future` 对象访问结果。

###### 设置

`set_value(val)`，设置结果为值，并使其处于就绪状态。

`set_value_at_thread_exit(val)`，同上，但当线程结束后，再使其处于就绪状态。

`set_exception(e_ptr)`，设置结果为异常，并使其处于就绪状态。

`set_exception_at_thread_exit(e_ptr)`，同上，但当线程结束后，再使其处于就绪状态。

###### 访问

`get_future()`，获取存储结果的 `future` 对象。













