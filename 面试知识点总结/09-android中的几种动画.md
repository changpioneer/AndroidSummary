
[参考](http://blog.csdn.net/yanbober/article/details/46481171)


Android中的几种动画：

曾被问到Android中有几种动画，这个问题也好难回答。Android3.0之前有2种，3.0后有3种。

1、FrameAnimation（逐帧动画）：将多张图片组合起来进行播放，类似于早期电影的工作原理，很多App的loading是采用这种方式。

2、TweenAnimation（补间动画）：是对某个View进行一系列的动画的操作，包括淡入淡出（Alpha），缩放（Scale），平移（Translate），旋转（Rotate）四种模式。

3、PropertyAnimation（属性动画）：属性动画不再仅仅是一种视觉效果了，而是一种不断地对值进行操作的机制，并将值赋到指定对象的指定属性上，可以是任意对象的任意属性。




