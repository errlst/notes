`QThreadPool` 借助 `QRunnable` 接口对线程池进行控制，每个 Qt 程序都有一个全局对象，通过静态函数 `globalInstance()` 访问。

#### 线程数控制

`maxThreadCount()`，`setMaxThreadCount(cnt)`，最大线程数量，默认为 `QThread::idealThreadCount()`。

`activeThreadCount()`，线程池中活动线程的数量，该值可能会大于最大线程数。

`reserveThread()`，保留一个线程，保留的线程不会分配给任务，且会让活动线程数+1。且无论保留多少个线程，线程池至少拥有一个可分配的线程。

`releaseThread()`，释放一个保留的线程。

#### 任务分配

`start(task, priority)`，保留一个线程并在其上运行任务，除非这个线程会使活动线程数大于最大线程数，则将其加入任务队列。

* _task_，可以是实现 `QRunnable` 接口的对象，也可以是任意接受零参数的函数对象。

#### 过期时间

`setExpiryTimeout(t)`，设置线程过期时间，在 _t_ 毫秒内未使用过的线程，将被销毁，并在需要时重新创建新线程。如果 _t_ 为负数，则无过期时间。默认为 30s。

`waitForDone(t)`，最多等待 _t_ 毫秒，让所有线程退出并销毁。如果 _t_ 为 -1，则忽略等待时间。