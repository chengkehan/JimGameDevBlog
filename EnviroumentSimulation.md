# 模拟昼夜及动态天气环境

**2016-10-6**

这次要实现的功能是在游戏中能表现出各种环境效果，比如清晨、正午、傍晚、夜晚、雨后、晚霞等等。第一个想到的办法是在每一个用到的着色器中加入相关的代码，使用宏开启或关闭。这种方式是最直接的，后来细想后发现这样做有很多问题：

* 对原来的着色器代码有一定的侵入性。
* 工作量是非常大的，用到的每一种着色器都要添加新的昼夜算法。
* 整体效果难以控制，每一种类型的的着色器的参数都各不相同。
* 各环境效果之间的过渡难以实现。

所以暂时放弃了使用这种实现方式。于是通过资料搜索，看到了 [Amplify Color](http://amplify.pt/unity/amplify-color/) 这个插件，这个插件使用一种非常巧妙的方式在对屏幕中色彩进行调整。在下面的内容中，我会解释下其实现的原理。在解释原理前，我们先看一下实现的测试效果，虽然这些效果在物理上并不是完全正确的，但基本达到了想表现出的效果。

> ![img](EnviroumentSimulation/1.gif =300x)
>
> 环境效果模拟测试。各效果之间的过度。

基于这个，可以在各天气环境中进行无缝过渡。

从这里开始，我们一步步说明下其中的基本原理。

> ![img](EnviroumentSimulation/6.png =600x)

这是一张基础色表（1024x32 像素），是 Amplify Color 的核心，表示在正常显示的情况下，图像的色表。可以很明显的看到整个色表是由一个个小色块组成，横向一共三十二个小色块，每个小色块的尺寸为 32x32 像素。我们从左往右看：

* 第 1 个小色块，其中所有像素的蓝色分量的值为 0，水平方向上红色分量值从 0 增加到 1，垂直分量上绿色分享的值从 0 增加到 1。右上角为黄色 RGB（1，1，0）。

* 第 2 个小色块，其中所有像素的蓝色分量的值为 $$$ 1 \over 31 $$$，水平方向上红色分量值从 0 增加到 1，垂直分量上绿色分享的值从 0 增加到 1。

* 第 3 个小色块，其中所有像素的蓝色分量的值为 $$$ { 1 \over 31 } \times 2 $$$，水平方向上红色分量值从 0 增加到 1，垂直分量上绿色分享的值从 0 增加到 1。

* ......以此类推

* 第 32 个小色块，其蓝色分量的值为 $$$ { 1 \over 31 } \times 31 $$$，水平方向上红色分量值从 0 增加到 1，垂直分量上绿色分享的值从 0 增加到 1。右上角为白色 RGB（1，1，1）。

通过这个规律可以看到，在屏幕后处理阶段，当从纹理中采样出颜色值后，先使用颜色值的蓝色分量来确定当前颜色值位于哪个小色块中，然后在用颜色值的红色和绿色分量来精确定位到小色块中的像素。如果这一切都是正确的话，定位到色表上的颜色值应该和纹理采样出的颜色值完全一致。如果我们对整个色表的颜色值进行调整，那么定位到的将不再是相同的值，而是我们调整后的值了。这就是其基本的原理。

> ![img](EnviroumentSimulation/7.png =600x)
>
> 夜晚的色表

那么如何对色表中的色彩进行调整呢？这里有一个基本的原则是，最终输出到新色表的颜色值只能依赖于基本色表中对应位置的颜色值。也就是说，比如新色表中位于（3，2）位置的颜色值，必须是从原始色表位于（3，2）的颜色值直接计算而来，不能依赖原始色表其他位置上的颜色值。这就是说，我们不能使用 Photoshop 中的滤镜对色表进行处理，但是可以使用 Photoshop 中**图像/调整（菜单）**下的所有功能对色表进行处理。

注意，这里存在一个问题，因为单个颜色通道能够表示 256 个值，而我们的小色块中只有 32 个值，显然差距很大，但是这个问题对最终的显示效果产生影响很小，几乎察觉不到。这也可能是 Amplify Color 为什么使用蓝色通道作为小色块索引的一个原因，因为自然界中物体色彩的蓝色分量相对较小。而红色绿色通道由于纹理采样时使用了双线性插值，所以也很难察觉到有异常。这些都是我个人的猜测。但是在某些情况下，还是可以看到这个瑕疵的，比如在一个色彩渐变的区域。在 Amplify Color 中，为了能够尽量缓解这个问题产生的副作用，还提供了一个高质量的渲染方案，就是对相邻小色块进行采样，然后对多个颜色量进行插值，当然这样会增加很多性能消耗。

通过 Amplify Color 我们可以做出很多炫酷的效果，对整体的画面色调进行调整，不仅仅是昼夜，任何色调的调整都是可以的，并且无缝平滑过渡。由于它是基于一个全屏的后处理效果，所以在性能上存在一定的消耗，但是这个消耗对于游戏整体效果的提升是值得的。知道了原理后就可以根据自己的需要做出最合适的方案了，其消耗是远远小于 Bloom 的。当然这种方法由于是全屏后处理，对屏幕上所有的像素都有效，我在实际使用的时候还增加了一张 RenderTexture 的 Mask，来控制色调映射的比例，这样对于一些不想影响到的像素区域可以忽略掉或者按一定比例进行混合。


