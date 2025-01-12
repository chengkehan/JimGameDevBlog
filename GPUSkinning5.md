# GPUSkinning 5

**2017-4-18**

[GPUSkinning V0.2](https://github.com/chengkehan/GPUSkinning) 版本的所有内容都已经完成，这个版本主要的工作是对上一个实验性质的版本的整理和总结，大部分基础代码都取自于上一个版本，新增的主要是编辑器相关的功能，这样使用上就更方便了。

一个比较大的变动是对骨骼动画进行采样的部分。前一个版本中是使用 Editor API 对 Animation 进行数据采样，但是这样做有点问题。首先因为使用到了 Editor API，所以只能在编辑器模式下工作，如果想将功能迁移到 Runtime 下，则无能为力了。其二，Unity 内部有多种动画系统，如果要做到兼容性，就必须编写不同的 Editor API 来对骨骼动画进行采样，还有可能当 Unity 新版本时要做出接口上的修改。基于以上这些原因，V0.2 版本使用了一种新的方法，在 Application 播放的状态下，通过代码控制动画跳转到指定点，然后记录下 Transform 信息。

另一个较大的变动是统计骨骼的方式。前一个版本中，使用 [SkinnedMeshRenderer.bones](https://docs.unity3d.com/ScriptReference/SkinnedMeshRenderer-bones.html) 这个 Unity 提供的 API 来获取所有骨骼，但是有时这个方法得到的骨骼并不完整。所以在 V0.2 版本中，改为手工指定一个根骨骼，采样时会迭代记录下其下的所有 Transform 数据，而不管这个 Transform 是不是骨骼（有可能是一个虚拟体）。这样就带来了一个额外的好处，不管是不是骨骼动画，只要导入 Unity 后能由 Animation 或 Animator 驱动 Transform 的，都可以被记录下来。

以上就是 V0.2 版本所作工作的大致说明。下一个 V0.2.1 版本，主要是让该工具的功能更为强大，TODO List 的内容主要包括：内存/对象管理上的优化，个体差异化动画，GPU Instancing，根骨骼动画，动画混合。

其实对于动画混合，我一直在考虑是否要去实现，因为这会增加很多 GPU 在 Skinning 时的开销，而最后开销的增加很可能会背离刚开始做这个工具的初衷，况且我也并不想为了功能而做功能。但是最后还是决定列入 TODO List 中，如果可行，最后该功能将会是一个可选方案，而非必选。当然不可能像 Blend Tree 那么强大，目前的设想是先实现类似 CrossFade，够用就好：）