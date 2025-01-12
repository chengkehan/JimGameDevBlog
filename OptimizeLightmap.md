# 优化 Lightmap

**2016-7-16**

我们通常会在移动设备上将光照信息烘焙到 Lightmap 中，避免昂贵的光照计算。当场景中模型非常多时，往往就面临着会烘焙出很多张 Lightmap，并且随着不断的提高烘焙质量，也会造成 Lightmap 数量上的增加。这里提出一种方法，希望可以解决这个问题。我们都知道通过调整 Scale In Lightmap 这个参数，可以让一些不重要的物件占用更少的 Lightmap 空间，顺着这个思路想下去，那么是否可以直接把 Scale In Lightmap 设置为 0 呢。表示该模型不被计入 Lightmap 中，但是会对周围计入 Lightmap 的物体产生影响（投影、间接光照等）。关于这一点在 Unity 的[官方文档中](http://docs.unity3d.com/460/Documentation/Manual/LightmappingInDepth.html)已经得到了确认，并且我经过测试也得到了正确的结果。既然 Scale In Lightmap 被设置为 0 的物体不被计入 Lightmap 中，那么如何让其接受光照呢。下面介绍方法。

在着色器中进行光照计算，具体怎么编写着色器代码，其实不用多解释，使用常规的光照计算方程即可。但是这种方法有一个缺点，就是无法实现间接光照和环境遮蔽的效果。如果没有这两种效果，大多数时候会显得很生硬，模型无法融入环境中。所以要配合另一种方法，就是利用顶点数据中的 color 值，来表示间接光照和环境遮蔽的效果。我们使用 color.rgb 来存储间接光照的颜色信息，使用 color.a 来存储环境遮蔽的信息。说到这其实已经非常明了了，只要在最终输出颜色之前，将 color.rgba 假象为间接光照和环境遮蔽，考虑进光照计算即可。原理很简单，关键是如何得到 color.rgba，最笨的办法就是手动绘制，写一个编辑器画笔工具即可，可是缺点是效率低下。所以通过查看相关的插件后得到一个可以自动生成间接光照和环境遮蔽数据的方法，虽然不是很精确，但对我来说足够好了。

方法是这样的。创建一个摄像机，摄像机会被逐一放置到顶点坐标相同的位置上，方向朝向法线方向，相机的 Field of View 和 Far Clip Plane 都是可以调整的（不同的值会影响到最终的效果），相机使用白色清屏，设置相机的 RenderTarget 为 RenderTexture（这张 RenderTexture 的尺寸不用很大，一般 64x64 就够了），对相机所看到的内容进行两次快照，第一次正常绘制所看到的内容，第二次是使用 Replacement Shader 将看到的内容绘制成黑色。对于第一张 RenderTexture，计算出其整体的色调值，这个值即为间接光照的颜色，为了让效果更好，我们还可以将顶点周围的光源考虑进来。对于第二张 RenderTexture，由于我们是使用白色清屏，黑色绘制，所以一般情况下部分像素为白色，部分像素为黑色，黑色像素占得比例即为环境遮蔽的程度，当然为了避免产生完全变黑的 AO，我们将最终的值映射到 0.4 到 1 之间，存储到 color.a 中。

这就是完整的方法了。注意一般情况下，这种方法所达到的效果是达不到 Lightmap 光照的效果的，模型的顶点数量太少了效果就不理想，所以要有选择的使用。所以对于一些场景边缘的模型，或者是非主要模型，都可以使用此方法。这样，对于这部分模型来说，就完全不会占用 Lightmap 的空间，多出来的 Lightmap 空间可以用到主要的模型上，让其精度更高，效果更好。


