#### 创建

通常有两种方式创建执行目标任务的线程：

* 继承 `QThread`，重写 `run()` 函数，并调用 `exec()` 进入事件循环。
* 使用静态函数 `create(f, args...)` 创建执行目标任务的线程。

线程对象创建后，还需调用 `start(pritority)` 开始执行线程。线程启动之前，会发送信号 `started()`。

```cpp
auto main(int argc, char *argv[]) -> int {
    auto app = QApplication{argc, argv};
    auto t = QThread::create([] {
        std::cout << "working\n"; // 2
    });
    QObject::connect(t, &QThread::started, [] {
        std::cout << "started\n"; // 1
    });
    t->start();
    return QApplication::exec();
}
```

#### 结束

`exit(code)`，告诉线程的事件循环退出，并返回代码。调用之后，线程从 `QEventLoop::exec()` 的调用返回。

`quit()`，等价 `exit(0)`。

`terminate()`，终止线程的执行，需要显示启动禁止，且具体行为和操作系统实现相关。

事件循环结束后，发出 `finished()` 信号，如果线程是通过 `terminate()` 结束，则通过其它线程发送 `terminate()` 信号。

#### 等待

`wait(deadline)`，阻塞当前线程，直到：

* 目标线程已经完成执行，或尚未开始执行。
* 到达截止时间。

#### 静态函数

`sleep(time)`、`msleep(msec)`、`usleep(usec)`，强制当前线程休眠一段时间。

`yieldCurrentThread()`，让出当前线程的执行。

`idealThreadCount()`，返回当前进程可以并行运行的理想线程数。

`currentThread()`、`currentThreadId()`，当前线程信息。