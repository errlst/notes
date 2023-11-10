[toc]

# thread

构造==thread==对象后，线程将立即进入OS调度。绑定到线程对象的函数，其返回值将被忽略，但如果其抛出异常终止，将调用==terminate()==。

## 析构

==thread==对象进行析构时，如果==.joinable() == true==，将调用==terminate()==。

即销毁之前，必须显示调用==.detach()==分离线程，或调用==.join()==等待线程结束。

## 线程id

使用==.get_id()==获取当前线程对象的id。对于所有执行线程，其id唯一；对于所有非执行线程，其共享一个唯一的id。

```cpp
auto main() -> int {
    auto t1 = std::thread{};
    auto t2 = std::thread{};
    std::cout << (t1.get_id() == t2.get_id()) << "\n";  // true

    return 0;
}
```

# jthread

==jthread==在==thread==的基础上增加了，析构时自动join、处理停止请求，功能。

## 构造

==jthread==的绑定函数，可接受==stop_token==作为第一个参数，其表示==jthread==对象的停止状态。

```cpp
using namespace std::chrono_literals;
auto main() -> int {
    auto t = std::jthread{
        [](std::stop_token tok) {
            while (!tok.stop_requested()) {
                std::cout << "work\n";
                std::this_thread::sleep_for(1s);
            }
            std::cout << "finish\n";
        }};
    std::this_thread::sleep_for(3s);
    t.request_stop();
    std::this_thread::sleep_for(3s);

    return 0;
}  // work*3 + finished
```

## 停止信号

==.get_stop_source()==，获取关联的==stop_source==对象。

==.get_stop_token()==，获取关联的==stop_token==对象。

==.request_top()==，向内部停止状态发送停止请求。

# this_thread

命名空间==this_thread==中定义了一些管理当前线程的函数。

==yield()==，建议交出当前线程的时间片。

==get_id()==，获取当前线程id。

==sleep_for(duration)==，休眠一段时间。

==sleep_until(point)==，休眠直到某个时间点。

# stop_token

==stop_token==维护一个停止状态，其可通过拷贝共享内部状态。

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

==stop_token==只提供观察该停止状态的方法，并不提供修改此状态的方法。通常并不直接使用==stop_token==，而是通过==stop_source==管理其内部维护的==stop_token==对象。

## 观察

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





