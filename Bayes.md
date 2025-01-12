# 模糊概率

**2023-5-20**

贝叶斯分析+模糊逻辑

经典概率计算，需要使用大量的样本进行分析，从而得到一个逼近最优的精确值。而贝叶斯分析，只需要少量的样本，因为它的目的不是计算出精确的值，而是要计算概率的分布。

这和我们大脑的思考方式很像。比如，当我们要去某个地方的时候，有三条路可以选择。我们在大脑中会根据自己的经验对道路的综合情况进行评估，来得到一个最初的路径规划，以便准时到达目的地。而我们并不能保证这种方案的路径规划一定会优于其它方案，也不会有一个精确值，只能说在某种程度上是相对较优的。

使用自己的生活经验对道路状况进行预测和评估，我们可以把这种行为称为贝叶斯分析中的先验概率。

这也是贝叶斯分析和经典概率之间最大的分歧。在经典概率里，绝对不会容许这种看似主观的东西存在，认为这是不科学的。而正是有了这个主观的先验概率，才使得贝叶斯分析的少量样本成为可能。

那么主观的先验概率是否会导致结果完全错误呢。相比传统的概率计算，贝叶斯分析并不是一步到位，直接计算出概率的具体结果，而是逐步优化概率分布。其实对于很多问题，也是没有办法直接使用经典的概率计算方式，得到一个具体的概率值的。

由于贝叶斯分析这种特性，导致每一次迭代的精确性宽容度相对较大。比如说，可以放弃一部分的精度来达到提升计算效率，这也就有了朴素贝叶斯的出现。即将各条件独立开，相互之间不会影响，这就导致各条件之间计算互不依赖，大大减少了计算量。

我们还是使用一个类似于上文中的例子来看看贝叶斯分析是如何在路径规划中起作用的。比如，在一个游戏中，我们需要向商队发送指令，将货物从城市A自动运送到城市B。显然从城市A到城市B不只有一条路径，假设有三条可选路径。那么在运送货物之前我们要考虑几点。首先，商队选择的道路是要通畅的，所谓的通畅指的是，最好是大路而不是颠簸的小路或山路，最好路径上没有敌对势力，最好路径上有驿站，最好路径是短的，等等。其次，由于商队运送的是食物，所以必须在指定的时间内到达，否则食物就会变质。

对于这个问题，我们人类的大脑基本已经有了自己的思路，我们可以模糊的描述出一个方案，但是现在我们需要将这些思路变为可以数据化的东西，以便让商队自动去处理。

一种可行的方法是，可以给每中情况标记一个 Rank 值，比如说，平坦的道路相比崎岖的山路会有更高的 Rank 值，路过盟友的城市相比经过一个危险地带会有更高的 Rank 值， 最后将每个 Rank 值相加，得分最高的路径胜出。

这种使用 Rank 值的方法好处是简单直接，而缺点是过于死板和绝对。因为算法是恒定的数值累加，最终一条路径计算出的总得分就会显得相对固定，缺少变数。而当每新增一个条件后，Rank 值的累加最大值就会增大，那么原先设置好的区间范围也需要随之调整，非常不便。并且在使用 Rank 值来判断是否能准时到达时也显得过于绝对。

所以这里我们使用的方法是，利用贝叶斯分析来搭出框架，使用模糊逻辑来填充这个框架。

由于每条路径的计算方法都是一样的，所以我们只看一条路径。

我们把第一条路径通畅的概率记为 P(A1)，把路径通畅且能够准时到达终点的概率记为 P(B|A1)，那么这条路径能够准时到达终点的概率为 P(A1)\*P(B|A1)。

P(A1) 的计算方法就要用到模糊逻辑，我们把每个条件都表示为一条模糊逻辑输入，最后将所有输入进行反模糊输出就能得到 P(A1)。这里有个很大的好处是，我们可以利用模糊逻辑将值域范围限定在 0 到 1 之间，这就非常有利于后续的分析和计算。

同理，商队走第二条路径，第三条路径的准时到达终点的概率为 P(A2)\*P(B|A2) 和 P(A3)\*P(B|A3)。

商队从起点准时到达终点的总概率为 P(A1)\*P(B|A1) + P(A2)\*P(B|A2) + P(A3)\*P(B|A3)。我们可以对每个概率使用模糊逻辑，决定哪条路更可行。同样，我们可以对总概率使用模糊逻辑，当模糊逻辑输出几乎不可能到达时，那么我们作为指挥官就要慎重考虑这次排遣指令了。

以上的分析是从为商队下达指令的玩家的角度来分析的，那么如果从敌对势力的角度来看。当敌方看到商队成功的完成了这次的重要的补集任务，导致对自己不利，敌方需要考虑的是，商队最有可能从哪条路过来的，在后续的 AI 处理中，就会在这优先条路上设置驻兵来阻截补给。

这时候贝叶斯公式就登场了。

从敌方的角度来看，商队选择第一条路径的概率为，P(A1|B) = P(A1)P(B|A1) / (P(A1)\*P(B|A1) + P(A2)\*P(B|A2) + P(A3)\*P(B|A3))

同理，可以推出商队选择第二条路径和第三条路径的概率。将这三个概率值输入模糊逻辑，再更具模糊逻辑的输出，来决定下一步 AI 的行为。





