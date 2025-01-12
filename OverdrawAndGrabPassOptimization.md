# 优化 Overdraw 和 GrabPass

**2016-10-17**

当进行大量的半透明 Fragment 渲染的时候就会产生大量的 Overdraw。究其原因，就是当渲染不透明 Fragment 的时候，可以根据深度缓冲中的数据将不必要的 Fragment 剔除掉，即使深度检测通过了，也只需要直接写入颜色数据即可。而渲染半透明 Fragment 时，无法通过深度缓冲剔除，而且需要读取已有的颜色，使其与新的颜色进行混合。这三点就造成了当绘制大量的透明物体时会导致严重的 Overdraw。

今天看到一种优化 Overdraw 方法，叫做 [OffScreenParticleRendering（离屏粒子渲染）][link1]。其原理是将会产生大量 Overdraw 的物体渲染到 RenderTexture 上，RenderTexture 的尺寸根据需要按照屏幕的分辨率进行缩小，由于分辨率小了，所以产生 Overdraw 的 Fragment 的量也大大减少了。需要注意的是，当渲染到 RenderTexture 上时，需要考虑到深度信息，因为透明的物体也有可能被不透明的物体遮挡住，最简单的方法就是渲染 RenderTexture 时使用 [_CameraDepthTexture][link2] 进行融合，就像做软粒子一样。最后将渲染好的 RenderTexture 叠加到主屏幕上。

而这种方法的一个问题是，由于 RenderTexture 的分辨率比主屏幕的分辨率小，所以会丢失细节，不适合用来做高频的渲染。所谓高频，就像是技能特效，色彩绚丽分明。而对应低频，就像是烟雾、迷雾、尘埃这一类，细节很少。另一个问题是，当部分需要用到离屏粒子渲染，而部分不需要时，就比较难处理了，因为这牵扯到渲染顺序的问题。最后，由于将 RenderTexture 叠加到主屏幕属于一个 PostEffect 操作，所以存在一定的消耗，这时候需要我们测试下，这种方法带来的好处是否大于一个 PostEffect 的消耗，否则就得不偿失了。如果条件允许，也可以不使用 PostEffect，而直接将 RenderTexture 绘制到一个 Plane 上。

[link1]: https://forum.unity3d.com/threads/released-off-screen-particles-render-particles-at-1-2-1-4-or-1-8-screen-resolution.358506/

[link2]: https://docs.unity3d.com/Manual/SL-CameraDepthTexture.html

下面一个优化是关于 [GrabPass][link3] 的。GrabPass 之所以消耗的，是因为当物体将要被渲染时抓取当前屏幕的内容，类似 glCopyTexSubImage2D 这样的大消耗方法。我们在使用 GrabPass 时大多都是进行扰动或者模糊处理，如果不仔细或者慢动作看，是分辨不出 GrabTexture 是不是精确的，所以并不需要一张实时的 GrabTexture。使用一个虚拟相机或使用 [CommandBuffer][link4] 更为合适。对于某些情况，甚至不需要每帧都做 Grab 操作。这样就能节省很多开销了。

[link3]: https://docs.unity3d.com/Manual/SL-GrabPass.html

[link4]: https://docs.unity3d.com/Manual/GraphicsCommandBuffers.html