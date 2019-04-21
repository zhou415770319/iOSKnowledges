##1.7离屏渲染

```
离屏渲染与光栅化
•离屏渲染：CPU渲染及非GPU缓冲区的渲染统称为离屏渲染。
•离屏渲染的触发：CoreGraphics的上下文绘制，drawRect绘制，layer圆角/边框/阴影/抗锯齿/光栅化（shouldRasterize置为YES）等。
•离屏渲染的检测：Instruments的CoreAnimation工具动态监测。（使用方法：Color Offscreen –Rendered Yellow ：开启后会把那些需要离屏渲染的图层高亮成黄色，黄色图层可能存在性能问题。）
•光栅化简介：隐式创建一个位图，各种阴影遮罩等效果也会保存到位图中缓存起来，从而减少渲染的频度，把GPU的操作转到CPU上，生成位图缓存，直接读取调用。（注：对于经常变动的内容，不要开启光栅化，防止性能浪费，如Cell的复用）
•光栅化的检测：Color Hits Green and Misses Red 开启后，若shouldRasterize设置为YES，对应的渲染结果会缓存，如果图层是绿色，表示缓存被复用；如果是红色，就表示缓存被重复创建，可能存在性能问题。
•GPU缓存区渲染优势：为图像显示做了高度优化，速度较快。
```