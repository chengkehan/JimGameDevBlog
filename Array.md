#数组
这里的数组指的是编程语言原生提供的一种数据结构，它是一种可以连续存储相同类型对象的容器，但是在开发的时候反而会较少用到，最大的原因就是没有直接提供常用的操作，比如Add、Insert、Remove、Clear等等，所以一般不会直接拿来使用。很多语言都会封装一套ADT（抽象数据结构），其中有些ADT的内部实现其事就是数组，所以我们如果能知道数组的优缺点的话，也就能很好的判断是否使用这种数据结构，并且根据需要来扬长避短。

需要给数组一个定义，上文中已经提到了，它是一种连续存储相同类型对象的容器。

首先是连续存储，这一点很重要，因为有很多ADT在内存中并不是连续存储的，这就会在内存中产生内存碎片，造成内存的浪费和效率低下。在程序实际执行的环境中，由一个专门的内存管理器来负责内存的申请、释放、整理。比如整个内存地址空间是一个大的仓库，那么内存管理器就是这个仓库的管理员，如果仓库里的货物没有归类放置整齐，而是分散在仓库的各个货架上，那么无论是想要统计、查找、迁出、整理获取都是非常困难的，会花费管理员很大的时间和精力，但是如果货物已开始就排放整齐，那么一切都会井然有序。内存碎片对于内存管理器也是一样的，它会严重影响到内存管理器的正常效率。

然后我们说一下第二点，相同的类型。再举个例子，我们有一百个小包裹需要装到一个大的箱子里，如果这些包裹大小不一，那么箱子里到底能装下多少个包裹呢，这是个很难回答的问题，因为每个包裹的大小都不一样，每个包裹的不同组合决定了是否最大限度的利用好箱子的空间。事实上我们有很多算法可以解决这个问题，但是这些算法都太费时了，在要求快速读写的内存上，显然不现实。但是如果每个包裹的大小都相同，那么问题就简单了，这样我们只需要往箱子里装包裹就行了，不需要去考虑其他的事情。

这样就把数组解释清楚了，那么它有什么优点和缺点呢。

优点就是它的连续存储，因为在内存中是连续的，相比一些非连续存储的数据结构，就会最大限度的减少碎片，碎片少了效率自然就高了，当然这其中牵扯到很多原因，比如说堆内存非配的机制、缓存的命中率等等，不在这里深究，有时间可以再查看下相关的资料。但是连续存储也是数组和其他数据结构比起来的一个最大的弱势，如果想在数组中间插入一个新的值，需要将这个插入点后的所有值向后移一格。就像排队时，如果在中间插进来一个人，那么这个人身后的所有人都需要往后退一步一样。如果需要退一步的人不多还好，但如果身后有成百上千个人，那么效果可想而知。

还有个很大优点就是相同的类型，也就是上文中说道的包裹的大小都一样。我们还是用排队来打比方，这里我们需要排成一个10行10列的方阵，不用说都知道两两之间的间距都是相等的，因为这样不但看这整齐，而且如果需要找到第4行第6列的那个人也非常的方面。但是如果两两的间距不相等的话呢，这样的方阵不但没法看，还没发管理，根本就不能算是个方阵。同样的，数组中每个元素的大小都是一样的，当我们需要找到第几个元素的时候会非常的方便和快捷。从这个排队的问题上就能直观的体会到。

最后说一个数组最大的缺点。就是数组一旦确定了最大容量就不能改变，还记得上面说的用来装包裹的箱子吗，箱子一旦制作好了就不会变大，如果设计成能够容纳50个包裹的箱子，那么就只能装得下50个包裹，多一个都不行。那么如果要多装怎么办呢，只能再拿个新的箱子了。所以有的数据结构就会对数组进行封装，你在往箱子里装包裹的时候，如果超过了最大的容量，这种数据结构就会自动的帮你换一个大的箱子，而你并不需要知道已经换了个大箱子这件事。这时候你就需要注意了，这种数据结构它真的是好心想要帮助你，但是如果经常进行这样的操作，那么或许会严重影响到效率。

为什么一开始就不用一个很大的箱子呢。大多数情况下却是是可以的，因为我们面临的问题大多都是可穷举的。

了解了数组后，才能更好的理解一些数据结构为什么会这么设计。