# GPUSkinning 4

**2017-3-12**

[GPUSkinning][link1] 新增骨骼绑点的功能，在演示场景中，在角色手部骨骼上挂载了武器。

> <img src="GPUSkinning4/1.gif" width="300"/>

在写这个功能的时候遇到了一些问题，在此记录下：

我没有使用 Unity 的 MeshRenderer 组件来渲染武器，而是使用 `Graphics.DrawMesh` 这个 API。一开始的考虑是避免创建太多的对象，造成不必要的开销，而后却发现，当大量的调用 `Graphics.DrawMesh` 这个 API 时，所产生的开销是非常大的。在这个演示中就是这样的情况，场景中有四百个角色，每个角色手中的武器都是通过 `Graphics.DrawMesh` 来绘制的，也就是说每帧都需要调用四百次 `Graphics.DrawMesh`，所以从开销上来说，使用 MeshRenderer 要更好。

由于武器并不是 SkinnedMeshRenderer，所以我是在 CPU 中计算武器所绑定的骨骼的变换矩阵的。大概是这样一个矩阵乘法：$ objectToWorld \cdot boneToObject \cdot localMat $。$ localMat $ 是武器以所在骨骼为原点的一个变换矩阵，因为武器并不一定正好绑定在骨骼点上，而是根据需求会有一定的偏移和旋转。$ boneToObject $ 是从要绑定的骨骼坐标系转换到角色模型坐标系的矩阵，这个矩阵并不需要计算，因为在做 GPUSkinning 时已经计算过了，直接缓存起来，供后续读取（缓存在 MatrixArray 或 Texture2D 中，具体可以看前几篇文章）。$ objectToWorld $ 是从角色模型坐标系转换到世界坐标系的矩阵，这个矩阵会经常发生变换，因为角色会移动、旋转。上文中说了，我使用 `Graphics.DrawMesh` 来渲染四百个模型，所以每一帧都需要在 CPU 中做四百次这样的矩阵乘法，这就给 CPU 带来了很多计算压力。同时我也试着使用 MaterialPropertyBlock，将这部分压力转移到 GPU 上，但是结果是四百次 `MaterialPropertyBlock.SetMatrix` 的开销同样很大。

在 Unity5.5 的版本中，加入了一个新的 API，是 `Graphics.DrawMeshInstanced`。这个 API 的好处是，在我明确知道自己要使用 [GPUInstancing][link2] 时，可以直接调用这个 API，这样就不需要像上文中所说的那样，调用四百次 `Graphics.DrawMesh`，并且可以使用 `MaterialPropertyBlock.SetMatrixArray` 将矩阵乘法的压力转移到 GPU 中。

使用 GPUInstancing 时，一个 Batch 的 Instance 数量是有上限的，上限的多少和当前设备的 [Uniform Buffer Object][link3] 最大容量有关。同时与 GPUInstancing 相关的还有 uniform array，比如在着色器中定义了 `uniform float4x4 _Matrices[n]`，使用 instanceID 来获取对应的数据，这里的 n 表示数组的长度，其值必须在编译时就确定下来，而不同硬件所支持的 uniform 数量的上限是不同的，在实际开发时要注意。

[link1]: https://github.com/chengkehan/GPUSkinning

[link2]: https://docs.unity3d.com/Manual/GPUInstancing.html

[link3]: https://www.khronos.org/opengl/wiki/Uniform_Buffer_Object
