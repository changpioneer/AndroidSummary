1、：说说Activity、Intent、Service以及他们之间的关系。 
A ：Activity负责界面的显示和用户的交互，Intent封装了数据，可以实现Activity之间以及Activity和Service之间数据的传递。Service运行在后台进程 ，一般我们会让给其运行一些后台任务 ，Activity通过StartService （Intent）或者BindService（Intent）可以启动Service。 


2、：请介绍一下ContentProvider是如何实现数据共享的。 
A ：我们可以定义一个类继承ContentProvider ，然后覆写该类的insert、delete、update等方法 ，在这些方法里访问数据库等资源。同时将我们ContentProvider注册在AndroidManifest文件中 ，其他应用需要使用的时候只需获取ContentResolver ，然后通过ContentResolver访问即可。 


3、你自己有做过自定义控件吗？ 
答 ：自定义控件做过，比如我们项目中的SlideMenu，LazyViewPager，Pull2RefreshListView， AutoScrollLoopViewPager，VerticalSeekbar,RandomLayout等都是自定义控件。 


4、ListView的优化你们是怎么做的？ 
答：ListView的优化有多种多样的策略。在我们的项目中主要做了如下优化。1、重用 

ConvertView ，2、给ConvertView绑定ViewHolder ，3、分页加载数据 ，4、使用缓存。
前两个是通用的解决方案 ，后两个是针对我们业务的个性化解决方案。我们的数据来自服务端 ，如果服务端有1000条数据的话，我们客户端不可能傻瓜式的一次性用 ListView把这些数据全部加载进来，因此我们就用分页加载数据，每次加载20页，当用户请求更多的时候再获取更多数据，网络的访问就算网速再快也多多少少会有一定的延迟 ，因此我们的网络请求是异步处理的，同时从网络加载来的数据使用了2级缓存来处理，第一级是内存级别的缓存，第二级是本地文件的缓存。当 ListView加载数据的时候首先从内存中找，如果找不到再去本地文件中找，只有都找不到的情况下才去请求网络。


5、异步加载网络数据


6、OOM异常怎么产生？是怎么解决的？ 
答：
(1)、图片过大导致OOM；
Android 中用 bitmap时很容易内存溢出。
  解决方法： 
   方法 1 ：等比例缩小图片
```java
 BitmapFactory.Options options = new BitmapFactory.Options(); 
        options.inSampleSize = 2; 
        //Options 只保存图片尺寸大小，不保存图片到内存 
        BitmapFactory.Options opts = new BitmapFactory.Options(); 
         opts.inSampleSize = 2; 
         Bitmap bmp = null; 
        bmp = BitmapFactory.decodeResource(getResources(), 
 mImageIds[position],opts); 
          //回收 
          bmp.recycle();// 
```
  以上代码可以优化内存溢出，但它只是改变图片大小，并不能彻底解决内存溢出。 

   方法2 ：对图片采用软引用，及时地进行recyle()操作 
```java
 SoftReference<Bitmap> bitmap = new SoftReference<Bitmap>(pBitmap); 
 if(bitmap != null){ 
     if(bitmap.get() != null && !bitmap.get().isRecycled()){ 
        bitmap.get().recycle(); 
        bitmap = null; 
     } 
  } 
```

(2)、界面切换导致OOM
有时候我们会发现这样的问题，横竖屏切换 N次后 OOM了。 
   这种问题没有固定的解决方法，但是我们可以从以下几个方面下手分析。 
   1、看看页面布局当中有没有大的图片，比如背景图之类的。 
   去除xml中相关设置，改在程序中设置背景图（放在onCreate()方法中）： 
```java
        Drawable drawable = getResources().getDrawable(R.drawable.id); 
        ImageView imageView = new ImageView(this); 
        imageView.setBackgroundDrawable(drawable); 

  在Activity onDestory()时注意，drawable.setCallback(null); 防止Activity得不到及时的释放。 
```
    2、跟上面方法相似，直接把 xml 配置文件加载成 view 再放到一个容器里，然后直接调用 this.setContentView(Viewview);方法，避免xml的重复加载。 
   3、 在页面切换时尽可能少地重复使用一些代码 
      比如：重复调用数据库，反复使用某些对象等等...... 

(3)、查询数据库没有关闭游标 
   程序中经常会进行查询数据库的操作 ，但是经常会有使用完毕Cursor后没有关闭的情况。如果我们的查询结果集比较小，对内存的消耗不容易被发现，只有在常时间大量操作的情况下才会出现内存问题，这样就会给以后的测试和问题排查带来困难和风险。
 
(4)、构造Adapter时，没有使用缓存的 convertView 
   在使用 ListView的时候通常会使用Adapter，那么我们应该尽可能的使用ConvertView。 

(5)、Bitmap对象不再使用时调用recycle()释放内存 
   有时我们会手工的操作 Bitmap对象，如果一个 Bitmap对象比较占内存，当它不再被使用的时候，可以调用Bitmap.recycle()方法回收此对象的像素所占用的内存，但这不是必须的，视情况而定。 
(6)、其他 
     Android应用程序中最典型的需要注意释放资源的情况是在Activity的生命周期中 ，在onPause()、onStop()、onDestroy()方法中需要适当的释放资源的情况。 


6.1、App在什么情况下会出现内存泄露？如何避免这些情况？ 
答：
       1）资源未及时释放，比如引用的io流资源、网络资源、数据库游标Cursor等没有释放；
       2）内存对象过大问题；保存了多个耗用内存过大的对象(如Bitmap、XML文件)，造成内存超出限制；Bitmap对象不使用时采用 recycle()释放内存；
       3）static关键字的使用问题；
static修饰的成员变量属于类，它的生命周期是很长的，如果用它来引用一些资源耗费过多的实例，比如Context，它会对Activity有引用，即使Activity已经onDestroy()，Context保存着Activity的引用，Activity也不会销毁，就会发生内存泄漏。
解决方案：
```
    应尽量避免static成员变量引用资源耗费过多的实例，如Context；

    使用WeakReference代替强引用。

    Context尽量只用ApplicationCo ntext，因为Application的Context的生命周期比较长，引用它不会出现内存泄漏的问题。
```

       4 ) 线程导致内存泄漏

线程产生内存泄露的主要原因在于线程生命周期的不可控。我们来考虑下面一段代码。 

 public class MyActivity extends Activity { 
     @Override 
     public void onCreate(Bundle savedInstanceState) { 
         super.onCreate(savedInstanceState); 
         setContentView(R.layout.main); 
         new MyThread().start(); 
     } 
     private class MyThread extends Thread{ 
 @Override 
         public void run() { 
             super.run(); 
             //do somthing while(true) 
         } 
     } 
  } 
假设 MyThread的run函数是一个很费时的操作，当我们开启该线程后，将设备的横屏变为了竖屏，一般情况下当屏幕转换时会重新创建Activity ，按照我们的想法，老的Activity应该会被销毁才对，然而事实上并非如此。 
   由于我们的线程是Activity的内部类，所以MyThread中保存了Activity的一个引用，当MyThread的run函数没有结束时，MyThread是不会被销毁的，因此它所引用的老的Activity也不会被销毁，因此就出现了内存泄露的问题。 

   有些人喜欢用Android提供的AsyncTask ，但事实上AsyncTask的问题更加严重 ，Thread只有在 run函数不结束时才出现这种内存泄露问题 ，然而AsyncTask 内部的实现机制是运用了ThreadPoolExcutor,该类产生的Thread对象的生命周期是不确定的 ，是应用程序无法控制的 ，因此如果AsyncTask作为Activity的内部类 ，就更容易出现内存泄露的问题。 
   针对这种线程导致的内存泄露问题的解决方案： 
      第一、将线程的内部类 ，改为静态内部类（因为非静态内部类拥有外部类对象的强引用，而静态类则不拥有）。 
      第二、在线程内部采用弱引用保存Context引用。 




7、那你们的图片是怎么处理的？  
答：图片的处理主要用两种方式。我们的应用中有两处用到了图片，一个是ListView中展 
示的图片缩略图，这种情况的特点是数量大，但是单个图片内存小，只有几kb，另外一种是大图片，就是用户通过手机拍摄的图片 ，然后通过http的post提交的方式提交到服务器上。然后在客户端将这个大图片也展示出来。对于第一种情形 ，我们是通过三种技术手段来解决问题的 ，一是图片的缓存策略，二是ListView的优化 ，其实在上面我已经讲过 ，三是WeakRefrence(弱引用)的使用。对于第二种情形，我们主要是首先通过 BitmapFactory.Options参数获取图片的宽和高 ，然后再根据我们ImageView的宽高对图片进行一个很大比例压缩。 


8、那你说说弱引用是怎么使用的？ 
答：WeakRefrence是一个类，在ArrayList中我们把这个类作为对象传递进去，把我们的图片放在WeakRefrence里面，这样当davlik虚拟机内存不够用的时候 ，就会把WeakRefrence对象回收掉，这样我们在WeakRefrence里面保存的数据也被回收了。 


9、我们的项目中有多个 Fragment，FragmentA跳转到 FragmentB，FragmentB跳转到 
FragmentC，那么这时候我多次按返回键，如何能让Fragment跟Activity的任务栈一样，依次从FragmentC跳转到 FragmentB，再跳转到FragmentA？
答：简单一点的话使用addToBackStack(String)将每个Fragment添加到回退栈；
    灵活一些的话，定义一个BaseFragment，让 FragmentA、FragmentB、FragmentC都继承 BaseFragment，在BaseFragment中定义一个ArrayList或者Map，每打开一个 Fragment，把这个 Fragment对象添加到ArrayList中，这样这个ArrayList就可以当做一个栈结构，第三我们需要设置返回键监听，当监听到返回键的时候，查看当前ArrayList中倒数第二个有没有 Fragment，如果有则取出该Fragment并把ArrayList中末尾Fragment删除，然后用 FragmentManager的Replace方法，将当前 Fragment替换成最新 Fragment即可，如果 ArrayList中只有一个Fragment，且监听到了返回键 ，那就不对Fragment做处理 ，同时也不拦截该事 
件，这样也不会影响其他Activity之间的切换。


10、AIDL的全称是什么？如何工作？能处理哪些类型的数据？ 
答： ①AndroidInterface DefinitionLanguage 
② AIDL一般用于远程服务 ，也就是进程间通信。我们可以分服务端和客户端 ，服务端声明AIDL文件，该文件命名为xxx.aidl，ADT会自动将xxx.aidl生成代码文件，代码文件提供了aidl中接口的实现。客户端如果要使用服务端提供的服务需要将xxx.aidl文件放到客户端源代码目录下，然后生成xxx.java类，客户端通过bindService的形参ServiceConnection的onServiceConnected获取到Service对象，这个对象通过Stub.asInterface （service）返回aidl的实现类。之后我们就可用调用这个aidl的实现类。 
③ 基本数据类型都可以，复杂对象也可以，只不过需要实现 Parcelable接口。  


12、java 如何调用c、c++语言？ 
答：java 通过JNI调用C/C++代码 ，在使用的时候首先通过System.loadLibrary("xxx")将xxx.so文件加载到jvm 中，同时在类中必须对so文件中的方法进行声明，格式:public native void test(); 


14、Android有哪些安全机制？ 
答：权限机制。我们的应用只要涉及到了用户的隐私、网络都需要在AndroidManifest.xml中进行声明，这样用户在安装的时候可以根据你申请的权限进行判断是否允许应用的某些行为。


15、Thread和AsyncTask的区别是什么？ 
答：AsyncTask是封装好的线程池 ，比起Thread+Handler的方式 ，AsyncTask在操作UI线程上更方便，因为onPreExecute()、onPostExecute()及更新UI方法onProgressUpdate()均运行在主线程中，这样就不用 Handler发消息处理了；


16、说说 MVC模式的原理，在Android中的运用。 
答：MVC是 Model、View、Controller三部分组成的。其中View主要由xml布局文件 ，或者用代码编写动态布局来体现。Model是数据模型 ，其实类似javabean ，不过这些JavaBean封装了对数据库、网络等的操作。Controller一般由Activity负责 ，它根据用户的输入 ，控制用户界面数据的显示及更新model对象的状态，它通过控制View和 Model跟用户进行交互。 


17、如何加载ndk库？如何在jni 中注册native函数，有几种注册方式？ 
答：通过System.loadLibrary("xxx")进行加载。
其实native有几种注册方式，自己当时并不知道，自己只知道一种注册方法，就是首先根据native 方法名，生成Java_com_xxx_MethodName(xxx,xxx);当然这个c/c++源码文件中需要引入jni.h ，然后把这个c/c++源码编译成so文件。自己后来百度了一下，网上有人数还有一种注册方式是动态注册，我就把关于动态注册的东西直接拷贝过来： 
JNI 允许你提供一个函数映射表 ，注册给Jave虚拟机 ，这样Jvm就可以用函数映射表来调用相应的函数 ，就可以不必通过函数名来查找需要调用的函数了。Java与JNI通过JNINativeMethod的结构来建立联系，它在jni.h 中被定义，其结构内容如下： 
```c
 typedef struct { 
       const char* name; //Java中函数的名字 
       const char* signature; //用字符串描述的函数的参数和返回值指向 函数的函数指针 
      void* fnPtr; // C 
     } JNINativeMethod; 
```
第一个变量name是Java中函数的名字。 
第二个变量signature ，用字符串是描述了函数的参数和返回值 
第三个变量fnPtr是函数指针，指向C函数。 
当java 通过System.loadLibrary加载完JNI动态库后 ，紧接着会查找一个JNI_OnLoad的函数 ，如果有，就调用它，而动态注册的工作就是在这里完成的。 
1)JNI_OnLoad()函数 
JNI_OnLoad()函数在VM执行System.loadLibrary(xxx)函数时被调用，它有两个重要的作用： 
指定JNI版本 ：告诉VM该组件使用那一个JNI版本(若未提供JNI_OnLoad()函数，VM会默认该使用最老的JNI 1.1版) ，如果要使用新版本的JNI ，例如JNI 1.4版，则必须由JNI_OnLoad()函数返回常量JNI_VERSION_1_4(该常量定义在jni.h 中) 来告知VM。 
初始化设定，当VM执行到System.loadLibrary()函数时，会立即先呼叫JNI_OnLoad()方法，因此在该方法中进行各种资源的初始化操作很恰当， 
2)RegisterNatives 
RegisterNatives在AndroidRunTime里定义 
syntax: 
jint RegisterNatives(jclassclazz,constJNINativeMethod* methods,jintnMethods)   


18、①在 Listview的优化中 ，我们为何使用ConvertView？②为何使用ViewHolder？③你认为哪个更能解决问题？④你认为view.inflate和view.findviewById哪个更耗时，为什么？⑤如果这两个AP让你重新写，你怎么写？ 
A ：①使用ConvertView可以实现对view的复用，这样大大节约了每次创建对象的时间，提升了ListView的显示效率。②使用ViewHolder作为内部类 ，可以将view的子控件封装在ViewHolder类中，然后通过View.setTag(ViewHolder)将view和ViewHolder进行绑定 ，这样我们就不用每次都调用view的findViewById(id)方法来查找控件。因为findViewById也是从布局文件中一个标签一个标签的找，也是降低效率的操作；③使用ConvertView解决了一大部分问题，使用ViewHolder实现了控件换时间的问题，因为给View对象设置一个Tag本身就是占用内存的，因此ViewHolder的使用还是需要区分不同的应用场景的 ，没有绝对的好与不好。如果内存足够需要高效则ViewHolder建议使用，否则不建议使用。
④当然是view.inflate耗时，这个函数完成的功能是把xml布局文件通过pullParser的形式给解析到内存中，需要io，需要递归子节点。
⑤我其实还不太相信我写出来的代码比Google官方写的好，如果让我写的话我可能会这样考虑，当用户在使用view.inflate的时候将多个id作为数组添加到形参中，这样在初始化view的使用我就可以给这个view直接调用setTag方法绑定需要的子控件。不过这个原生方法其实也应该保留，以供不同的需求使用。 
















 