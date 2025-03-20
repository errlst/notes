无栈协程：不保留调用堆栈信息。

每个 coroutine 都和以下内容关联：

- promise object：在协程内部进行操作，协程通过它提交结果或异常。

  > promise object 和 `std::promise` 没有任何关系。

- coroutine handle：在协程外部进行操作，用于恢复协程执行或销毁协程。

  > coroutine handle 不具有协程的所有权，只负责指向一个具体的协程对象并操作。

- coroutine state：协程内部动态分配的存储器对象，存储：

  - promise object。

  - 函数参数。

  - 当前挂起点。

  - 生命周期跨越当前挂起点的局部变量和临时变量。

# 启动

协程执行时：

1. 通过 `operator new` 分配 coroutine state。

2. 拷贝函数参数到 coroutine state。

   > 值传递参数复制或移动，引用传递参数保存引用。

3. 调用 promise object 构造函数，且传递拷贝后的函数参数。

   > ```cpp
   > struct promise_type;
   >
   > struct task_t : public std::coroutine_handle<promise_type> {
   >     using promise_type = ::promise_type;
   > };
   >
   > struct promise_type {
   >     promise_type(int &i) { i = 1; }
   >
   >     auto get_return_object() -> task_t { return {task_t::from_promise(*this)}; }
   >     auto initial_suspend() noexcept -> std::suspend_never { return {}; }
   >     auto final_suspend() noexcept -> std::suspend_never { return {}; }
   >     auto return_void() -> void {}
   >     auto unhandled_exception() -> void {}
   > };
   >
   > auto foo(int i = 0) -> task_t {
   >     std::println("{}", i); // 1
   >     co_return;
   > }
   >
   > auto main() -> int {
   >     foo();
   >     return 0;
   > }
   >
   > ```

4. 调用 `promise.get_return_object()` 并在局部变量中保存返回结果，当协程第一次挂起时将结果返回给调用者。

   > 此步骤在内及之前，异常都会传播到调用者，而不是保存在 promise。

5. 执行 `co_await promise.initial_suspend()`。典型的 promise 返回 `std::suspend_always` 以延迟启动协程，或者返回 `std::suspend_never` 立即启动协程。

   > 恢复后，开始执行协程主体。

# 结束

## 异常

协程因为未捕获异常结束时：

1. 捕获异常并调用 `promise.unhandled_exception()`。

2. 执行 `co_await promise.final_suspend()`。

## co_return

协程执行到 `co_return expr` 时：

1. 调用：

- `promise.return_void()`，如果 epxr 为空或 `void`。

- `promise.return_value(expr)`，否则。

2. 按创建的相反顺序销毁自动存储周期变量。

3. 执行 `co_await promise.final_suspend()`。

## 销毁

协程因为 co_return、异常或通过句柄销毁时：

1. 调用 promise 析构函数。

2. 调用拷贝的函数参数的析构函数。

3. 调用 `operator delete` 释放 coroutine state。

4. 执行权转移到 caller/resumer。

# co_await

`co_await expr` 作为值表达式可以出现在通常函数体内（包括 lambda），但不能出现在：

- `catch` 代码块中。

- 默认参数。

- 具有静态或线程存储周期的块变量的初始化器中。

表达式的执行为：

1. 将 expr 转换为 awaitable：

   - 如果 expr 被 `initial_suspend()`、`final_suspend()` 或 `yield expr` 创建，那 awaitable 就是 expr。

   - 如果表达式 `promise.await_transform(expr)` 良构，则 awaitable 是该表达式。

   - 否则，awaitable 就是 expr。

2. 获取 awaiter：

   - 如果重载决策可以给出最佳决策，那么 awaiter 是 `awaitable.operator co_await()` 或者 `operator co_await(static_cast<awaitable_t&&>(awaitable))` 的结果。

   - 如果不存在 operator await，那么 awaiter 就是 awaitable。

   - 否则，ill-formed。

3. 调用 `awaiter.await_ready()`，如果结果可转换为 `false`，则协程被挂起，调用 `awaiter.await_suspend(handle)`。

   > handle 是当前协程句柄。

   - 如果返回 `void` 或 `true`，控制权立即转移到 caller/resumer。

   - 如果返回 `false`，恢复当前协程。

   - 如果返回其他协程句柄，则恢复其他协程。

     > 通过调用 `handle.resume()`。

   - 如果抛出异常，则捕获异常，恢复协程，并重新抛出异常。

     ```cpp
     struct promise_t;

     struct task_t : public std::coroutine_handle<promise_t> {
        using promise_type = ::promise_t;
     };

     struct promise_t {
        promise_t(int &i) { i = 1; }

        auto get_return_object() -> task_t { return {task_t::from_promise(*this)}; }
        auto initial_suspend() noexcept -> std::suspend_never { return {}; }
        auto final_suspend() noexcept -> std::suspend_never { return {}; }
        auto return_void() -> void {}
        auto unhandled_exception() -> void {}
     };

     struct await_excception_t : std::suspend_always {
        auto await_suspend(std::coroutine_handle<>) -> void {
           throw 0;
        }
     };

     auto foo(int i = 0) -> task_t {
        try {
           co_await await_excception_t{};
        } catch (...) {
           std::println("catch exception");
        }
     }

     auto main() -> int {
        foo();
        return 0;
     }
     ```

4. 调用 `awaiter.await_resume()`，且结果为 co_await 表达式的结果。

# co_yield

`co_yield expr` 等价于 `co_await promise.yield_value(expr)`。
