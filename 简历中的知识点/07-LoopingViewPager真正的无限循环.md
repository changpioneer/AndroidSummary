    


传统的ViewPager的实现逻辑是自定义一个View继承ViewPager,在适配器中将count() 设置为Integer.MAX_VALUE.这种情况,指示在理论上很难达到边界,并非真正意义上的无限广播轮播.
    LoopingViewPager实现了真正意义上的无限循环。
例如，原来的适配器创建的条目是4条[0,1,2,3]，修改的适配器将会创建6条[0,1,2,3,4,5]。如下代码：
```java
@Override
    public int getCount() {
        return mAdapter.getCount() + 2;
    }
```

这6条与之前的4个条目的对应关系用toRealPosition()方法实现，
[0->3, 1->0, 2->1, 3->2, 4->3, 5->0]

```java
int toRealPosition(int position) {
        int realCount = getRealCount();
        if (realCount == 0)
            return 0;
        int realPosition = (position-1) % realCount;
        if (realPosition < 0)
            realPosition += realCount;
​
​
        return realPosition;
    }
```
