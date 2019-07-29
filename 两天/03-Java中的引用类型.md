##一、Java中的引用类型（**）

1. Toast的操作也属于UI的修改，因此必须在主线程中（近似正确）。




3. Java中的引用类型（不属于课程范畴）

- 强引用 （使用=号连接起来的引用） 当一个对象被强引用引用到的时候，GC永远也无法回收
- 软引用 （SoftReference）当系统内存不足的时候被GC干掉
- 弱引用 （WeakRefrence）当GC发现到他的时候就被干掉
- 虚引用 （PhantomRefrence）主要用于对象是否被GC回收的检测工作


4. GC垃圾回收算法之1：引用计数算法（废弃的算法），对象每引用一次，该对象的计数器+1，如果计数器为0，则视为垃圾，可以被回收。


##二、引用的案例
如果str="abc"，则调用gc()时不能清除弱引用，因为GC回收的是堆内存的垃圾，"abc"在常量池中。

ReferenceQueue，当GC回收str,WeakReference对str的引用断开时，WeakReference被放进ReferenceQueue。通常被用来检测内存泄漏。

```java
//LeakedCanary检测内存泄漏的框架
public class ReferenceTest {
    public static void main(String[] args) {
        //GC回收内存中的垃圾,堆内存中的垃圾
        //强引用
        String str = new String("abc");
        //软引用
        SoftReference<String> softReference = new SoftReference<>(str);
        //弱引用
        ReferenceQueue<String> referenceQueue = new ReferenceQueue<>();
        WeakReference<String> weakReference = new WeakReference<>(str, referenceQueue);
        
        //清除强引用
        str = null;
        System.out.println("str:"+str);
        
        //清除软引用，在内存不足的时候被回收
        softReference.clear();
        System.out.println("softReference:"+softReference.get());
        
        //清除弱引用，被GC发现就回收
        System.gc();
        System.out.println("weakReference:"+weakReference.get());
        
        //当GC回收str,weakReference对str的引用断开时，weakReference被放进ReferenceQueue
        //从ReferenceQueue中拿出weakReference
        Reference<? extends String> reference = referenceQueue.poll();
        System.out.println("reference:"+reference);
    }
}
```


