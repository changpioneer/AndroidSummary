
####构造方法
```java
     public AsyncTask() {
        //实际上就是创建了一个Runnable
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                ...
                return postResult(doInBackground(mParams));
            }
        };
​
​
        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                ...
            }
        };
    }
```


####核心方法execute()

```java
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec, Params... params) {
       
​
​
        mStatus = Status.RUNNING;//切换状态
​
​
        onPreExecute();
​
​
        mWorker.mParams = params;//接收参数
        exec.execute(mFuture);//线程池执行任务
​
​
        return this;
}
```


#### mFuture就是Runnable的子类
```java
  public void run() {
            ...
       
            Callable<V> c = callable;//callable就是构造mFuture时传入的mWorker
            //mWorker是Callable的子类，核心方法就是call()
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();//调用call()方法
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
       
    }
```


####call()方法一调用就调用到了mWorker的call()中
    
```java
mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
​
​
                mTaskInvoked.set(true);//设置该任务是否已经被调用
​
​
                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //执行doInBackground中的任务
​
​
                return postResult(doInBackground(mParams));
            }
        };
```


####postResult将结果发送到主线程
   
```java
 private Result postResult(Result result) {
        //发送一个消息，消息内容就是当前AsyncTask的实例对象及任务的结果
        Message message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
```



#### AsyncTaskResult包装结果对象及当前的AsyncTask对象
  
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

####处理消息
   
```java
 private static class InternalHandler extends Handler {
        
        public void handleMessage(Message msg) {
            AsyncTaskResult result = (AsyncTaskResult) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    //result.mTask就是当前的AsyncTask对象
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
```

#### 调用finish()方法
   
```java
private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);//将任务结果发布出去
        }
        mStatus = Status.FINISHED;
    }
```



####发布任务进度
   
```java
  protected final void publishProgress(Progress... values) {
        if (!isCancelled()) {
            //通过发消息的方式，将进度发送到主线程处理
            sHandler.obtainMessage(MESSAGE_POST_PROGRESS,
                    new AsyncTaskResult<Progress>(this, values)).sendToTarget();
        }
    }
```


####处理进度的消息
  
```java
public void handleMessage(Message msg) {
            AsyncTaskResult result = (AsyncTaskResult) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    //result.mTask就是当前的AsyncTask对象
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    //调用更新进度的方法
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
```
