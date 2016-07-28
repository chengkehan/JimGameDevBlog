# 昼夜模拟

**2016-7-?**

这次要实现的功能是在游戏中能有昼夜的效果。第一个想到的办法是在每一个用到的着色器中加入相关的代码，使用宏开启或关闭。这种方式是最直接的，后来细想后发现这样做有很多问题：

* 对原来的着色器代码有一定的侵入性。
* 工作量是非常大的，用到的每一种着色器都要添加新的昼夜算法。
* 整体效果难以控制，每一种类型的的着色器的参数都各不相同。

所以暂时放弃了使用这种实现方式。于是通过资料搜索，看到了 [Amplify Color](http://amplify.pt/unity/amplify-color/) 这个插件，这个插件使用一种非常巧妙的方式在对屏幕中色彩进行调整。在下面的内容中，我会解释下其实现的原理。在解释原理前，我们先看一下实现的效果，虽然这些效果在物理上并不是完全正确的，但基本达到了想表现出的内容（当然也是我对色彩的理解有限，未能调出最好的效果）。

白天 | 夕阳 
------------ | ------------- 
![img](DayNightSimulation/2.jpg =400x) | ![img](DayNightSimulation/3.jpg =400x)

渐黑 | 全黑 
------------ | ------------- 
![img](DayNightSimulation/4.jpg =400x) | ![img](DayNightSimulation/5.jpg =400x)

> ![img](DayNightSimulation/1.gif =300x)
>
> GIF 动态图。各效果之间的过度。

这里我最不满意的是云彩部分，由于这是一张截图，所以无法对云彩进行单独控制（将云彩过度为星空）。实际应用到项目中时应该会做的更好。

---

> 注：原图是来自**上古卷轴5**的场景截图。

---

从这里开始，我们一步步说明下其中的基本原理。

> ![img](DayNightSimulation/6.png =600x)

这是一张基本的色条（1024x32 像素），表示在正常显示的情况下，图像的色彩。可以很明显的看到整个色条是由一个个小色块组成，横向一共三十二个小色块，每个小色块的尺寸为 32x32 像素。我们从左往右看：

* 第 1 个小色块，其蓝色分量的值为 0，水平方向上红色分量值从 0 增加到 1，垂直分量上绿色分享的值从 0 增加到 1。右上角为黄色 RGB（1，1，0）。

* 第 2 个小色块，其蓝色分量的值为 $$$ 1 \over 31 $$$，水平方向上红色分量值从 0 增加到 1，垂直分量上绿色分享的值从 0 增加到 1。

* 第 3 个小色块，其蓝色分量的值为 $$$ { 1 \over 31 } \times 2 $$$，水平方向上红色分量值从 0 增加到 1，垂直分量上绿色分享的值从 0 增加到 1。

* ......以此类推

* 第 32 个小色块，其蓝色分量的值为 $$$ { 1 \over 31 } \times 31 $$$，水平方向上红色分量值从 0 增加到 1，垂直分量上绿色分享的值从 0 增加到 1。右上角为白色 RGB（1，1，1）。

通过这个规律可以看到，当从纹理中采样出颜色值后，先使用颜色值的蓝色分量来确定当前颜色值位于哪个小色块中，然后在用颜色值的红色和绿色分量来精确定位到小色块中的像素。如果这一切都是正确的话，定位到色条上的颜色值应该和纹理采样出的颜色值完全一致。如果我们对整个色条的颜色值进行调整，那么定位到的将不再是相同的值，而是我们调整后的值了。这就是其基本的原理。

> ![img](DayNightSimulation/7.png =600x)
>
> 夜晚的色条

那么如何对色条中的色彩进行调整呢？这里有一个基本的原则是，最终输出到新色条的颜色值只能依赖于基本色条中对应位置的颜色值。也就是说，比如新色条中位于（3，2）位置的颜色值，必须是从原始色条位于（3，2）的颜色值直接计算而来，不能依赖原始色条其他位置上的颜色值。这就是说，Photoshop 中的各种滤镜都是不能使用的，但是可以使用 Photoshop 中**图像/调整（菜单）**下的所有功能。

注意，这里存在一个问题，因为单个颜色通道能够表示 256 个值，而我们的小色块中只有 32 个值，显然差距很大，但是这个问题不会对最终的显示效果产生影响，我猜测这和硬件的纹理过滤模式有关，硬件会自动对采样的周围像素值取平均（Bilinear Filter）。

通过 Amplify Color 我们可以做出很多炫酷的效果。由于它是基于一个全屏的后处理效果，所以在性能上存在相当的消耗，但是我觉得这个消耗对于游戏整体效果的提升是值得的。

