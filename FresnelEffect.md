#Fresnel 效果

**2016-3-24**

在 Shader 中，Fresnel 描述的是随着观察角度角度变化，在特定角度所观察到的反射变换关系。一般我们在写材质的时候会在表面平滑的物体上使用 Fresnel 效果，比如水面，这样能提高画面的真实感。在基于物理的材质上都会有这个参数。一开始对这个参数不是很了解，通过查看了一些资料渐渐的开始知道了其中的意思，感觉 Fresnel 有点是介于高光和漫反射之间的，但更偏向高光，同样也是在特定的角度（观察视角的物体边缘，反射周围环境），但是能量没有高光集中。

下面是一些参考资料，在此记录下：

[http://filmicgames.com/archives/557](http://filmicgames.com/archives/557)
[http://filmicgames.com/archives/547](http://filmicgames.com/archives/547)
[http://kylehalladay.com/blog/tutorial/2014/02/18/Fresnel-Shaders-From-The-Ground-Up.html](http://kylehalladay.com/blog/tutorial/2014/02/18/Fresnel-Shaders-From-The-Ground-Up.html)
