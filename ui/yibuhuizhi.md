##1.6异步绘制原理

```
怎么进行异步绘制呢,其实就是基于系统给我们开的口子layer.delegate,如果遵从或者实现了displayLayer方法,我们就可以进入到异步绘制流程当中,在异步绘制的过程当中
就由delegete去负责生成bitmap位图
设置改bitmap作为layer.content属性的值
```
通过一副时序图来了解异步绘制的机制和流程
![](/media/editor/异步绘制_20190418102924880063.jpeg)
