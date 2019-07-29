
```java
public abstract class AsyncTaskEx<Params, Progress, Result> {
​
​
    private static final int corePoolsize = 2;
    private static final int maximumPoolSize = 5;
    private static final long keepAliveTime = 60L;
    private static BlockingQueue<Runnable> workQueue = new LinkedBlockingDeque<>(100);
    static ThreadPoolExecutor sThreadPoolExecutor = new ThreadPoolExecutor(corePoolsize,
            maximumPoolSize, keepAliveTime, TimeUnit.SECONDS, workQueue, new ThreadFactory() {
        AtomicInteger count = new AtomicInteger();
        @Override
        public Thread newThread(Runnable r) {
            return new Thread(r,"AsyncTaskEx-sThreadPool"+count.incrementAndGet());
        }
    });
​
​
    private Runnable mRunnable;
​
​
    private Handler mHandler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
        }
    };
​
​
    public AsyncTaskEx(){
        mRunnable = new Runnable() {
            //需要执行的任务
            @Override
            public void run() {
                //在子线程中执行
                final Result result = doInBackground(mParamses);
                //将任务的结果发布出去
                mHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        onPostExecute(result);
                    }
                });
            }
        };
    }
​
​
    private Params[] mParamses;
    public AsyncTaskEx<Params, Progress, Result> execute(Params...paramses) {
        this.mParamses = paramses;
        //任务执行前调用
        onPreExecute();
​
​
        //在线程池中执行该子线程
        sThreadPoolExecutor.execute(mRunnable);
        return this;
    }
​
​
    protected void onPreExecute() {
    }
​
​
    protected abstract Result doInBackground(Params... params);
​
​
    protected void onProgressUpdate(Progress... progresses) {
    }
​
​
    protected void onPostExecute(Result result) {
    }
​
​
    //发布进度，是在子线程执行
    protected final void publishProgress(final Progress... progresses) {
        mHandler.post(new Runnable() {
            @Override
            public void run() {
                //更新进度，在主线程执行
                onProgressUpdate(progresses);
            }
        });
    }
}
```

在构造方法中创建子线程；

- 当执行execute()方法时，获取传递进来的参数，执行onPreExecute()，然后将该子线程放进线程池并执行；
- 在该子线程中执行doInBackfround()方法，执行完成后将onPostExecute()方法通过Handler.post()方法抛到主线程执行；
- publishProgress()是在子线程中执行的，是将进度信息传递到onProgressUpdate()方法中，所以在publishProgress()中使用Handler.post()将onProgressUpdate()抛到主线程中执行。





