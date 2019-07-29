1、AsyncTask

2、IntentService

3、Loader

4、JobScheduler and GcmNetworkManager

5、CountDownTimer

6、Java Threads or Android HandlerThread

7、FutureTask

8、Java Timer / ScheduledThreadPoolExecutor


AsyncTask: 为 UI 线程与工作线程之间进行快速的切换提供一种简单便捷的机制。适用于当下立即需要启动，但是异步执行的生命周期短暂的使用场景。

HandlerThread: 为某些回调方法或者等待某些任务的执行设置一个专属的线程，并提供线程任务的调度机制。

ThreadPool: 把任务分解成不同的单元，分发到各个不同的线程上，进行同时并发处理。

IntentService: 适合于执行由 UI 触发的后台 Service 任务，并可以把后台任务执行的情况通过一定的机制反馈给 UI。