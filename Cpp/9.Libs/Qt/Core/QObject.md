`QObject` 是 Qt 对象模型的核心，该模型的核心功能是称为信号槽的对象通讯机制。使用 `connect` 将信号连接到槽，`disconnect` 销毁连接，或者使用 `blockSignals()` 暂时阻止信号。受保护函数 `connectNotify` 和 `disconnectNotify` 可以用于追踪连接。

## 对象树

使用另一个对象作为父对象创建新的 `QObject` 时，会将新对象添加到父对象的子对象列表中，当父对象析构时，会自动销毁其所有子对象。

对于栈上分配的局部对象，其析构时会从父对象的对象列表中摘除；而在堆上动态分配的对象，父对象析构时会自动 `delete`。

`children()` 返回子对象列表，新添加的子对象为列表的最后一个元素。

`parent()` 返回父对象。

## MOC

对所有继承自 `QObject` 的类，都应该使用 `Q_OBJECT` 宏，且在该源文件上运行 moc 编译器。

在 cmake 中，可以开启 `CMAKE_AUTOMOC`，构建时会扫描目标源上的头文件，且对存在 `Q_OBJECT` 宏的头文件自动处理生成对应源文件，并链接。

## 信号槽

信号槽机制是类型安全的，槽的签名必须匹配信号的签名，但槽的签名可以短于信号签名，多余的参数会被忽略。在进行信号槽连接时，可以使用函数指针让编译器检查类型是否匹配，也可以使用 `SIGNAL` 和 `SLOT` 宏在运行时进行基于字符串的类型判断。

```cpp
class sender : public QObject {
    Q_OBJECT
  signals:
    auto sig(int, double) -> void;
};

class receiver : public QObject {
    Q_OBJECT
  public slots:
    auto slot(int) -> void { std::cout << "slot\n"; }
};

auto main(int argc, char *argv[]) -> int {
    auto app = QApplication{argc, argv};
    auto s = sender{};
    auto r = receiver{};
    QObject::connect(&s, &sender::sig, &r, &receiver::slot);
    QObject::connect(&s, SIGNAL(sig(int, double)), &r, SLOT(slot(int)));
    emit s.sig(0, 0);
    return QApplication::exec();
}
```

### 绑定重载的槽函数

Qt 中绑定重载的槽函数有以下几种方式：

- 使用 `QOverload<func_arg>::of(&func)`，选择重载的槽函数，Qt5.7+ 可使用。

- 使用 `static_cast<func_type>(&func)`，不推荐。

- 使用 lambda，通用。

### 连接

`QObject::connect(sender, signal, receiver, slot, type)`，创建从发送者对象发送的信号，到接受者对象的槽的连接。且可以将一个信号连接到另一个信号，进行信号间的传播。

- _type_，确定信号和槽之间的连接方式。
  - `AutoConnection`，如果 _receiver_ 处于发送信号的相同线程，行为为 `DirectConnection`，否则，行为为 `QueuedConnection`。
  - `DirectConnection`，信号发出后，在同一线程立即调用槽函数。
  - `QueuedConnection`，控制流程返回 _receiver_ 线程的事件循环时，调用槽函数。
  - `BlockingQueuedConnection`，同上，但信号线程会阻塞直到槽函数返回。如果 _receiver_ 和发送信号在同一线程，会导致死锁。
  - `UniqueConnection`，一个组合标志，如果同一信号已经连接到同一接受者的同一槽，连接失效。
  - `SingleShotConnection`，一个组合表示，该连接只会完成一次，调用槽函数后自动断开。

### 断连

`QObject::disconnect(sender, signal, receiver, slot)`，断开连接，可以使用 `nullptr` 表示任何信号、任何接受者或任何槽。通常有以下三种方式使用 `disconnect`：

- `disconnect(sender, nullptr, nullptr, nullptr)`。

  断开与对象信号连接的所有连接。

- `disconnect(sender, signal, nullptr, nullptr)`。

  断开连接到对象特定信号的所有连接。

- `disconnect(sender, nullptr, receiver, nullptr)`。

  断开到特定接受者的所有连接。

`blockSignals(block)`，阻塞该对象发出的信号，但即使信号被阻塞，`destroy()` 信号也会发出。

### 发送者

如果槽是成员函数，可以通过 `sender()` 获取发送者。如果该对象与发送者在不同线程，该函数返回值无效。

## 事件

Qt 主事件循环从事件队列中获取本机窗口系统事件，将其转换为 `QEvents`，并发送到 `QObject`。通常，事件来自底层窗口系统，但也可以通过 `QCoreApplication::sendEvent()` 或 `QCoreApplication::postEvent()` 主动发送事件。

### 事件处理

虚函数 `event(e)` 接受发送到当前对象的事件，并进行处理。如果事件被识别并处理，应该返回 `true`；如果返回 `false`，事件会在对象树中进行传播。

除了通过 `event()` 接受事件，Qt 提供的类都提供了一些虚函数，当某些事件传送到该对象后，自动调用这些事件处理函数。

### 事件过滤

使用 `installEventFilter(filter)` 在对象上安装事件过滤器。过滤器可以是任意继承自 `QObject` 的类，在事件传送到对象之前，会调用过滤器的 `eventFilter(watched, e)` 函数，如果其返回 `true`，则该事件不会传输给该对象。

## 关联线程

每个 `QObject` 都与一个线程关联，该线程是其事件处理的上下文，如果一个 `QObject` 对象无关联线程，其就无法进行事件处理。

`QObject` 对象创建时，其会自动关联到创建该对象的线程，可以使用 `thread()` 获取关联的线程。

### 修改关联

如果当前对象没有父对象，且当前线程是该对象的关联线程，可以使用 `moveToThread(target)` 修改改对象关联的线程。

修改关联时，会将该对象的所有活动计时器暂停，然后在目标线程中以相同的间隔重新启动。因此如果在线程之间不断移动对象，可能会无限推迟计时器事件。

```cpp
auto t1 = std::unique_ptr<QThread>{};
auto t2 = std::unique_ptr<QThread>{};

auto main(int argc, char *argv[]) -> int {
    auto app = QApplication{argc, argv};
    auto timer = QTimer{};
    QObject::connect(&timer, &QTimer::timeout, []() {
        std::cout << "finish\n";
        QApplication::exec();
    });
    t1.reset(QThread::create([&] {
        while (true) {
            if (timer.thread() == QThread::currentThread()) {
                std::cout << "t1 move to t2\n";
                timer.moveToThread(t2.get());
            }
        }
    }));
    t1->start();
    t2.reset(QThread::create([&] {
        while (true) {
            if (timer.thread() == QThread::currentThread()) {
                std::cout << "t2 move to t1\n";
                timer.moveToThread(t1.get());
            }
        }
    }));
    t2->start();
    timer.start(1);
    timer.moveToThread(t1.get());

    return QApplication::exec();
}
```

## 定时器

### 启动

`QObject` 可以直接作为定时器使用，`startTimer(interval, type)` 注册定时器，成功返回定时器标识符，失败返回 0。

- _interval_，如果间隔为 0，那么每当没有更多时间需要处理时，就产生一次定时器事件。

- _type_，定时器精确度类型。

### 注册

`killTimer(id)` 终止计时器。

### 接受

计时器事件发送 `QTimerEvent`，默认会调用虚函数 `timerEvent(event)`。
