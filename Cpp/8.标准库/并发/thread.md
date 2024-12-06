

#### thread

###### 构造

`thread(f, args...)`，创建 `thread` 对象并将其和执行线程关联，新的线程将立即进入os调度中，且执行`invoke(auto{std::forward<F>(f)}, auto{std::forward<Args>(args)}...)`。

```cpp
struct T {
    T() = default;
    T(const T &) { std::println("const T&"); }
    T(T &&) noexcept { std::println("T&&"); }
};

auto main() -> int {
    auto t = T{};
    std::thread{[](T t) {}, t}.join();
    // const T&     传递构造参数时会进行拷贝构造
    // T&&          新线程执行时会进行移动构造
    return 0;
}
```

`auto` 求值产生的异常会在抛到当前线程，而不会开始新线程。执行线程的返回值会被忽略，且其抛出异常将直接调用 `terminate()`。

###### 析构

`thread` 对象销毁前，必须调用 `.detach()` 分离线程，或调用 `.join()` 等待线程结束，否则在析构时，如果 `.joinable() == true`，将调用 `terminate()`。

###### id

`.get_id()` 获取当前线程对象的 id。所有执行线程的 id 唯一，所有非执行线程共享一个 id。

```cpp
auto main() -> int {
    auto t1 = std::thread{};
    auto t2 = std::thread{};
    std::println("{}", t1.get_id() == t2.get_id()); // true
    return 0;
}
```

---

#### this_thread

`this_thread::yield()`，向内核提醒可以重新调度线程的执行。

`this_thread::get_id()`，获取当前线程 id。

`this_thread::sleep_for(dur)`，休眠一段时间。

`this_thread::sleep_until(point)`，休眠到某个点。

---

#### jthread

`jthread` 的一般行为同 `thread` 一致，但其会在析构时自动合并，且可在特定情况下停止。

###### 构造

传递给 `jthread` 的执行函数，可额外接受 `stop_token` 作为第一个参数，其表示绑定对象的停止状态。

```cpp
using namespace std::chrono_literals;
auto main() -> int {
    auto t = std::jthread{[](std::stop_token tok) {
        while (!tok.stop_requested()) {
            std::println("work");
            std::this_thread::sleep_for(1s);
        }
        std::println("end");
    }};
    std::this_thread::sleep_for(3s);
    t.request_stop();
    std::this_thread::sleep_for(3s);
    return 0;
}
/// 输出：work*3 + end
```

###### 停止信号

`.get_stop_resource()`，获取关联的 `stop_source` 对象。

`.get_stop_token()`，获取关联的 `stop_token` 对象。

`.request_top()`，发送停止请求。

---

#### stop_token

`stop_token` 提供观察其内部停止状态的功能，可通过拷贝共享内部状态。通过 `stop_resource` 管理其内部关联的 `stop_token` 对象。

```cpp
auto main() -> int {
    auto source = std::stop_source{};
    auto stop = source.get_token();
    std::cout << stop.stop_requested() << "\n";  // false
    source.request_stop();
    std::cout << stop.stop_requested() << "\n";  // true
    return 0;
}
```

###### 观察

==.stop_requested()==，检查是否已经请求停止。

==.stop_possible()==，检查其是否可以请求停止。拥有关联停止关系的token都可请求停止，即使其已处于停止状态。

```cpp
auto main() -> int {
    auto source = std::stop_source{};
    std::cout << std::stop_token{}.stop_possible() << "\n";   // false
    std::cout << source.get_token().stop_possible() << "\n";  // true
    source.request_stop();
    std::cout << source.get_token().stop_possible() << "\n";  // true

    return 0;
}
```

# stop_source

==stop_source==内部管理其绑定的==stop_token==对象。

## 修改

==.request_top()==，对关联停止状态发出停止请求。

## 观察

==.get_token()==，获取其关联停止状态。

==.stop_request()==，略。

==.stop_possible()==，略。

# stop_callback

==stop_callback==对象绑定==stop_token==对象和回调函数，当发出停止请求时，将调用回调函数[^1]。

```cpp
auto main() -> int {
    auto source = std::stop_source{};
    auto stop_cb = std::stop_callback{
        source.get_token(), [] {
            std::cout << "stop callback\n";
        }};
    source.request_stop();  // stop callback

    return 0;
}
```

==stop_callback==对象可以提早销毁，其等价于注销回调函数。

```cpp
auto main() -> int {
    auto source = std::stop_source{};
    std::stop_callback{
        source.get_token(), [] {
            std::cout << "stop callback\n";
        }};
    source.request_stop();  // 无输出

    return 0;
}
```

[^1]:只能触发一次





