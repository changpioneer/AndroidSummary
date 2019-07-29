
###1. Service的两种启动方式&生命周期（***）
* startService ： 启动的Service一直运行，已经与Activity的生命周期没有任何关系。 
* bindService ： 启动的Service的生命周期和Activity就绑定了，如果Activity销毁了，那么Service也销毁了（onUnbind->onDestory）
* startService和bindService ： 会一直运行，如果Activity销毁了，Service只会调用onUnbind方法，但是并不会销毁。

###2. Activity和Service（bound）的交互方式（***）
* IPC：Inner Process Communication 进程间通信。

####3.1 Extending the Binder class（同一进程中）  
####3.2 Using a Messenger
####3.3 Using AIDL（不同进程之间）
AIDL使用步骤：

服务端(支付宝)

- 定义AlipayService 继承Service，在service中定义了真正的支付方法pay（）；
- 定义一个Alipay.aidl文件（先写一个接口类，然后将后缀名改为.aidl，同时将所有的权限修饰符给去掉），系统会自动生成一个Alipay.java；
- 在该.aidl文件中定义一个pay（）接口方法，参数与service中定义的pay（）方法一样；
- 在Service中定义一个类AlipayBinder extends Alipay.Stub
- 覆写pay方法，在覆写pay方法中调用AlipayService 的pay()方法；
- 在onBind(Intent)中返回AlipayBinder对象
- 在AndroidManifest.xml中进行注册，需要添加intent-filter

客户端(美团)

- 将Alipay.aidl拷贝到美团工程src目录下，注意：包名必须严格一致！
- 系统会自动生成一个Alipay.java
- 调用bindService（intent, alipayServiceConnection, Service.BIND_AUTO_CREATE）,传递的是隐式意图
- 创建AlipayServiceConnection implements ServiceConnection
- 覆写onServiceConnected和onServiceDisconnected方法
- 在onServiceConnected(ComponentName name, IBinder service)方法中，service是AlipayService 返回的Binder对象，使用如下方法获取Alipay对象
            Alipay alipay = Alipay.Stub.asInterface(service);
- 之后调用alipay中的方法即可


##Binder通讯机制：

- 创建一个MusicService 继承Service，在该Service中实现播放音乐的真正逻辑；
- 自定义一个MusicPlayerBinder 继承 Binder类；
- 覆写onBind方法，在onBind方法中返回实现IBinder接口MusicPlayer ；
- 在MusicPlayerBinder类中提供声明为public的方法，在这些方法中，通过MusicService.this.调用Service中对应的方法；
- Service 需要在AndroidManest.xml中进行注册；

- 在MainActivity中，通过bindService（Intent，ServiceConnection，flag）方法，绑定MusicService；
- 在MainActivity中，定义一个MusicServiceConnection 实现ServiceConnection接口；
- 覆写该接口中的两个方法，主要是覆写该方法：onServiceConnected（name,IBinder）；
- 在onServiceConnected（name,IBinder)方法中，IBinder就是MusicService 的onBind()方法返回的IBinder接口MusicPlayer，将IBinder对象强转为MusicPlayer；
- 之后通过调用MusicPlayer对象的方法，就可以实现跟Service直接的调用过程。

```java
public class MusicService extends Service {
    @Override
    public IBinder onBind(Intent intent) {
        MusicPlayer musicPlayer = new MusicPlayer();
        return musicPlayer;
    }   
    public void play(String songName){
        Log.d("tag", "播放："+songName);
    }   
    public void stop(){
        Log.d("tag", "音乐播放器停止。");
    }   
    public int getProcess(){
        int process = new Random().nextInt(100);
        Log.d("tag", "当前播放进度："+process);
        return process;
    }   
    class MusicPlayer extends Binder{       
        public void play(String songName){
            MusicService.this.play(songName);
        }       
        public void stop(){
            MusicService.this.stop();
        }       
        public int getProcess(){
            return MusicService.this.getProcess();
        }
    }
}
```

```java
public class MainActivity extends Activity {
​
​
    private MusicServiceConnection musicServiceConnection;
    private MusicPlayer musicPlayer;
​
​
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        //通过startService启动一次Service，让Service在后台一直运行
        startService(new Intent(this, MusicService.class));
    }
    //绑定音乐服务
    public void bindMusicService(View view){
        Intent service=new Intent(this, MusicService.class);
        musicServiceConnection = new MusicServiceConnection();
        /**
         * 参数1：Intent对象，用于找到Service，显示或者隐式
         * 参数2：服务绑定成功或者意外终止时的回调对象
         * 参数3：要绑定的Service的启动策略通常为单一的值：BIND_AUTO_CREATE
         */
        bindService(service, musicServiceConnection, Service.BIND_AUTO_CREATE);
    }
    
    class MusicServiceConnection implements ServiceConnection{
        /**
         * 参数1：组件名称（Service）
         * 参数2：MusicService 的onBind方法返回的对象（MusicPlayer）
         */
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.d("tag", "ComponentName="+name+"///IBinder="+service);
            Toast.makeText(MainActivity.this, "服务已经绑定，可以播放音乐了。", Toast.LENGTH_SHORT).show();
            /**
             * 将IBinder对象强转为MusicPlayer对象
             */
            musicPlayer = (MusicPlayer)service;
        }
        /**
         * 当Service意外终止回调该方法
         */
        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.d("tag", "服务意外终止了"+name);
            Toast.makeText(MainActivity.this, "服务意外终止了"+name, Toast.LENGTH_SHORT).show();
        }       
    }
    //播放音乐
    public void playMusic(View view){
        
        if (musicPlayer==null) {
            Toast.makeText(this, "音乐服务还未绑定，请先绑定再操作！", Toast.LENGTH_SHORT).show();
            return;
        }
        musicPlayer.play("只妈妈露笑脸");
    }
    //获取音乐播放的进度
    public void getMusicProcess(View view){
        if (musicPlayer==null) {
            Toast.makeText(this, "音乐服务还未绑定，请先绑定再操作！", Toast.LENGTH_SHORT).show();
            return;
        }
        int process = musicPlayer.getProcess();
        Toast.makeText(this, "当前音乐的播放进度为："+process, Toast.LENGTH_SHORT).show();
    }
    //停止音乐播放
    public void stopMusic(View view){
        if (musicPlayer==null) {
            Toast.makeText(this, "音乐服务还未绑定，请先绑定再操作！", Toast.LENGTH_SHORT).show();
            return;
        }
        musicPlayer.stop();
        
        if (musicServiceConnection!=null) {
            //解绑Service
            unbindService(musicServiceConnection);
            musicServiceConnection = null;
            musicPlayer = null;
        }
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (musicServiceConnection!=null) {
            //解绑Service
            unbindService(musicServiceConnection);
            musicServiceConnection = null;
            musicPlayer = null;
        }
    }
}
```
