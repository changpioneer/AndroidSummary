


1：访问其他应用程序的Activity

如调用系统通话应用

```java
IntentcallIntent=newIntent(Intent.ACTION_CALL,Uri.parse("tel:12345678");
startActivity(callIntent);
```

2：Content Provider

    如访问系统相册

3：广播（Broadcast）

    如显示系统时间

4：AIDL服务

[aidl的使用和Binder通讯机制](../简历中的知识点/01-aidl的使用和Binder通讯机制.md)



4种跨进程通讯的方式：Activity、ContentProvider、Broadcast和AIDL Service。其中Activity可以跨进程调用其他应用程序的Activity；ContentProvider可以访问其他应用程序返回的 Cursor对象；Broadcast采用的是被动接收的方法，也就是说，客户端只能接收广播数据，而不能向发送广播的程序发送信息。AIDL Service可以将程序中的某个接口公开，这样在其他的应用程序中就可以象访问本地对象一样访问AIDL服务对象了。

http://blog.csdn.net/yan8024/article/details/6444368





