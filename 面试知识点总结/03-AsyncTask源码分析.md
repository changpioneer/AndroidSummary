
AsyncTask的缺陷，可以分为两个部分说，在3.0以前，最大支持128个线程的并发，10个任务的等待。在3.0以后，无论有多少任务，都会在其内部单线程执行；

###1、先看构造方法。

```java
public AsyncTask() {
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //noinspection unchecked
                    result = doInBackground(mParams);
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                    mCancelled.set(true);
                    throw tr;
                } finally {
                    postResult(result);
                }
                return result;
            }
        };
​
​
        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }
```

别看AsyncTask的构造方法只是简单的创建了WorkerRunnable和FutureTask对象，但是这两个对象很重要。我们先来看看WorkerRunnable是什么？

```java
private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
        Params[] mParams;
    }
```

这就是WorkerRunnable的全部代码了，里面只有一个Params数组。关键是它实现了Callable接口，这个很关键，我们来看看Callable的代码：

```java
public interface Callable<V> {   
    V call() throws Exception;
}
```

可以看到Callable接口里面只定义了一个call方法，返回一个泛型对象。这么短的几行代码，为什么说它重要呢？因为不论是继承Thread类还是实现Runnable方法，执行完任务以后都无法直接返回结果。而Callable接口就弥补了这个缺陷，当call方法执行完毕以后会返回一个泛型对象。WorkerRunnable实现了Callable接口，也就要实现该方法。该方法中的内容我们暂时先不看。



在AsyncTask的构造方法中也创建了FutureTask对象并重写了done方法，这个方法的内容我们也暂时不看。先了解一下FutureTask这个类。FutureTask实现了RunnableFuture<V>接口，而RunnableFuture又同时继承了Runnable和Future<V>接口。Runnable接口我们很了解，其中只有一个run方法，并且无返回值。Future<V>接口主要是针对Runnable和Callable任务的。
提供了三种功能：
1.判断任务是否完成
2.能够中断任务
3.能够获取任务执行的结果
FutureTask同时实现了Runnable和Future接口，也就是说FutureTask既能够被线程执行，又能提供线程执行任务后返回的结果。对于Futrue的介绍就先到这，之后我们会接着看。




##2、接着就执行AsyncTask中的execute方法了。

```java
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }
```

我们可以看到在方法内部调用了executeOnExecutor()方法并传入了sDefaultExecutor对象，我们来看看sDefaultExecutor对象的定义。

```java
public static final Executor SERIAL_EXECUTOR = new SerialExecutor();
​
​
private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
​
​
private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;
​
​
        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }
​
​
        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```

sDefaultExecutor实际上就是SerialExecutor对象，SerialExecutor实现了Executor接口，在SerialExecutor内部定义了一个双端队列ArrayDeque，ArrayDeque的内部是使用数组形式来实现双端队列的，我们知道队列是FIFO的，只能在队头删除元素，队尾添加元素，而双端队列是在队头和队尾都能够删除和添加元素。需要注意的是ArrayDeque没有容量的限制，队列满了以后会自动进行扩充。关于ArrayDeque的相关知识可以自行谷歌。对于sDefaultExecutor我们暂时只需要知道它是一个SerialExecutor对象就行。


##3、我们接着看executeOnExecutor()方法:

```java
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }
        mStatus = Status.RUNNING;
        onPreExecute();
        mWorker.mParams = params;
        exec.execute(mFuture);
        return this;
    }
```

方法很简单，首先判断当前的状态，如果是在运行当中或者已经任务结束都会抛出异常。接着就会执行onPreExecute方法，是不是很眼熟？没错就是我们在DownloadTask中重写的方法，我们当时的注释是在doInBackground前执行，现在我们知道这个方法在什么时候被回调了吧？接着把我们传入进来的参数（也就是url）赋值给mWorker.mParams，mWorker我刚刚已经看了其定义，内部有一个Params的数组来存放传入的参数。接着我们就使用exec.execute方法了，这里的exec就是我们刚刚讲的SerialExecutor对象，现在调用它的exeute方法并把我们在AsyncTask构造方法中创建的mFuture对象传入，还记得mFuture对象吧？它实现了Runnable接口。

```java
private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;
​
​
        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }
​
​
        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```

同样的代码，这次我们得一步步来看。在execute方法接收一个Runnable对象，我们传入的mFuture正好实现了该接口，方法内部首先通过双端队列ArrayDeque的offer方法把一个新创建的Runnable对象入队。然后判断mActive是否为空？第一次执行肯定是为空的，所以会调用scheduleNext方法，scheduleNext方法首先从双端队列中得到一个元素，如果不为空，就调用THREAD_POOL_EXECUTOR.execute方法，THREAD_POOL_EXECUTOR是一个线程池，也就是说我们把从双端队列中得到的任务交给线程池去执行（接下来的代码都在工作线程中执行），那么肯定就要执行到Runnable对象的run方法，该方法中的内容就是调用mFuture中的run方法:

```java
public void run() 
 {
        if (state != NEW ||
            !U.compareAndSwapObject(this, RUNNER, null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
```

该run方法为FutureTask类中的run方法，不要搞混淆了。这个方法我们不细讲，我们只看重点，方法中的callable就是我们在AsyncTask构造方法中创建FutureTask时传入的mWorker对象，我们知道mWorker是实现了callable接口的。然后调用了mWorker的call方法：



##4、 doInBackground();

```java
mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //noinspection unchecked
                    result = doInBackground(mParams);
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                    mCancelled.set(true);
                    throw tr;
                } finally {
                    postResult(result);
                }
                return result;
            }
        };
```

还记得这段代码吗？这是AsyncTask构造方法中创建WorkerRunnable对象的代码。我们看它的call方法。首先将mTaskInvoked这个标志位设置为true，然后设置了线程优先级，接着我们看见在这里回调了doInBackground方法，所以doInBackground方法是在子线程中执行的，doInBackground方法中会有一个返回值，这个返回值会传入postResult方法中。

```java
private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
```

postResult中的逻辑就是获取一个Message，然后发送该Message给Handler处理。我们看sHandler的定义：

```java
private static final InternalHandler sHandler = new InternalHandler();
private static class InternalHandler extends Handler
    {
        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg)
        {
            AsyncTaskResult result = (AsyncTaskResult) msg.obj;
            switch (msg.what)
            {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
```

这是sHandler在AsyncTaskResult中的定义，很显然它是在主线程中定义的。在Loop从MessageQueue中获取到Message后会分发到handleMessage这个方法中来处理，我们可以看见在Switch代码块中有两个分支，一个是用来处理结果的，一个是用来更新进度的。我们先看处理结果的分支，处理结果的分支会调用AsyncTask中的finish方法并把任务的结果传入。

##5、onPostExecute(result)

```java
 private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }
```

finish方法中的逻辑很简单，判断用户是否调用了cancel方法，如果调用了那么就回调onCancelled方法，如果没有调用就回调onPostExecute方法，这跟我们在讲AsyncTask基本用法时说的逻辑一样。最后标志任务状态为完成。（注意：以上代码是在主线程中执行）



##6、publishProgress()

我们接着看更新进度的分支，我们先想一下更新进度时在什么时候被调用？是不是在doInBackground中调用了publishProgress方法？所以我们先看看publishProgress方法：

```java
protected final void publishProgress(Progress... values) {
        if (!isCancelled()) {
            getHandler().obtainMessage(MESSAGE_POST_PROGRESS,
                    new AsyncTaskResult<Progress>(this, values)).sendToTarget();
        }
    }
```

从publishProgress方法我们知道在doInBackground中调用了publishProgress方法传入值后，如果任务没有被取消，那么就会发送一个Message，接着就回到handleMessage方法中的处理进度代码块，回调onProgressUpdate方法，这时候我们就可以对进度条进行更新。

我们可以看见在使用sHandler获取一个Message时都传入了一个AsyncTaskResult对象，我们看看它的定义：

```java
private static class AsyncTaskResult<Data> {
        final AsyncTask mTask;
        final Data[] mData;
​
​
        AsyncTaskResult(AsyncTask task, Data... data) {
            mTask = task;
            mData = data;
        }
    }
```

AsyncTaskResult只是简单对AsyncTask和返回结果做了封装。


http://www.jianshu.com/p/e60c3bb03d61 
