##1、Bundle

四大组件中的三大组件(activity、service、BroadcastReceive)都是支持Intent中传递Bundle数据的，由于Bundle实现了Parcelable接口，所以他可以方便地在不同的进程间传输。基于这一点，当我们在一个进程中启动另一个进程的activity、service、BroadcastReceive，我们就可以在Bundle中附加我们需要传输给远程进程的信息并通过Intent发送出去。

##2、文件共享

文件共享方式要考虑并发读写的问题。文件共享方式适合在对数据同步要求不高的进程之间进行通信，并且要妥善处理并发读写问题。

SharedPreferences也属于文件的一种，但是由于系统对他的读写有一定的缓存策略，即在内存中会有一份SharedPreferences文件的缓存，因此在多进程模式下，系统对它的读写就变得不可靠，当面对高并发的读写访问，SharedPreferences有很大的几率会丢失数据。

##3、AIDL

##4、Messenger

Messenger是一种轻量级的IPC方案，它的底层实现是AIDL。



##5、ContentProvider

##6、Socket




#使用多进程的问题

1、静态成员和单例模式完全失效；

2、线程同步机制完全失效；

3、SharedPreferences的可靠性下降；

4、Application会多次创建。