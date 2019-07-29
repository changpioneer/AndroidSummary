###1.优缺点
- Thread+Handler 代码比较多
- Thread+runOnUiThread 代码结构不是很好
- AsyncTask是一个轻量级的线程封装工具。分过程使用的。任务开始前(准备工作) 任务中(更新进度)任务结束(提示)

###2.使用Thread通常是以下几个环节和AsyncTask对应的方法
- loading 视图加载                                                  onPreExecute() 
- run   处理耗时代码                                               doInBackground(Params... params) 
- 在子线程中发消息handler.sendMessage               publishProgress() + onProgressUpdate() 
- 处理子线程发的消息hanldeMessage                     onPostExecute(Result result)


###3.示例代码：

```java
//第二个参数 publishProgress提交参数的类型 publishProgress-->onProgressUpdate
        //第一个参数 输入值输execute --> doInBackground 
        //第三个参数 返回值  doInBackground-->onPostExecute
        AsyncTask<String, Integer, File> task = new AsyncTask<String, Integer, File>(){
            /**
             * 任务前的准备
             */
            @Override
            protected void onPreExecute() {
                super.onPreExecute();
                progressDialog = new ProgressDialog(BackupSmsActivity.this);
                progressDialog.setTitle("短信备份中...");
                progressDialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
                progressDialog.setCancelable(true);//使返回键无法关闭对话框
                progressDialog.show();
            }
            private int progress = 0;
            /**
             * 执行耗时代码
             */
            @Override
            protected File doInBackground(String... params) {
                for (int i = 0; i < 100; i++) {
                    try {
                        Thread.sleep(100);
                        progress += 1;
                        publishProgress(progress);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                File file = new File("mnt/sdcard/song.mp3");
                return file;
            }
            /**
             * 这是主线程，接受子线程通过publishProgress(values)发过来数据
             */
            @Override
            protected void onProgressUpdate(Integer... values) {
                super.onProgressUpdate(values);
                int progress = values[0];
                progressDialog.setMax(100);
                progressDialog.setProgress(progress);
            }
            /**
             * 任务完成
             */
            @Override
            protected void onPostExecute(File result) {
                super.onPostExecute(result);
                System.out.println(result);
                Toast.makeText(getApplicationContext(), "任务完成了 刷新界面", 0).show();
                progressDialog.dismiss();
            }
        };
        //执行任务
        task.execute("http://www.baidu.com/mp3/song.mp3");
```

task.execute("~~~")开始执行任务，先在方法onPreExecute()进行准备工作，然后执行doInBackground()，task.execute("~~~")传入的是字符串，所以onPreExecute()的参数是String类型；doInBackground()方法中下载数据，查询数据等等，通过publishProgress()方法将值传递给onProgressUpdate()方法，传递的值是什么类型，onProgressUpdate()的参数就是对应的包装类型，onProgressUpdate()是在主线程执行的任务；doInBackground()执行完成后，给onPostExecute()返回一个值。
 