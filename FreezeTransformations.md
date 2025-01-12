#Maya 中 Freeze Transformations 的作用

**2016-2-18**

Maya 中可以对物体进行 Reset Transformations 和 Freeze Transformations 两种操作。Reset Transformations 是将物体的变换重置到原点状态，物体会移动到零点，旋转归零，缩放为一。那么 Freeze Transformations 呢，查了些资料，是不改变物体的位置、旋转、缩放，将物体的当前变换设置为原点状态，也就是说物体当前的位置变为零点，当前的旋转角度为零旋转。

那么使用 Freeze Transformations 有什么好处呢。当对一个物体进行了大量变换操作后，我们可以对这个物体进行一次 Freeze Transformations 操作，以后如果想撤销对该物体后续的变换操作，并还原到 Freeze Transformations 这个点。只需要将变换重置为零即可，而不需要去记录这个还原点时刻的变换数值。

还有就是 Joint，当调整完 Joint 的位置、旋转角度后，Joint 的局部坐标的朝向会变得指向四面八方，很不规则，这对于后续编辑 IK 时会造成困难和影响。对 Joint 对使用 Freeze Trasnformations 然后在使用 Orient Joint，将 Joint 的局部坐标朝向统一，方便后续操作。