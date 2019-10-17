## 序

1968年，的一篇论文，“P. E. Hart, N. J. Nilsson, and B. Raphael. A formal basis for the heuristic determination of minimum cost paths in graphs. IEEE Trans. Syst. Sci. and Cybernetics, SSC-4(2):100-107, 1968”。从此，一种精巧、高效的算法------A*算法横空出世了，并在相关领域得到了广泛的应用。

## 启发式搜索算法

启发式搜索算法就是当前搜索节点在选择下一步节点时，可以用过一个启发函数进行计算代价，选择代价最小的节点作为下一步搜索节点

## A*估值函数

$$f(n) = g(n) + h(n)$$

* f(n):从初始状态经由状态n到目标状态的代价估计
* g(n):是在状态空间中从初始状态到状态n的实际代价
* h(n):是从状态n到目标状态的最佳路径的估计代价

>路径搜索问题中，状态就是节点，代价就是距离

## 运行机制

* 使用两个状态表Open和Clodes。Open表由待考察的节点组成，Closed表由已经考察过的节点组成
* 当一个节点已经被检查过所有与它相邻的节点，计算出了这些节点的f(n)，并把其放入Open表，那么这个节点就放入Closed表
* 开始时，Closed表为空，Open表仅包含起始节点，每次迭代中，将Open表中最小代价的节点进行检查，如果不是目标节点，则进行以下处理：
    1. 如果相邻节点不在两个表中，则将其插入Open表中
    2. 如果相邻节点在Open表中，则对比新旧路径的代价值并取最低的存入Open表
    3. 如果相邻节点在Close表中，则对比新旧路径的代价值，如果新路径更低，则移出Closed表，加入Open表
* 重复以上步骤，如果到达目标节点前，Open表为空，则意味着没有可达的路径

## 寻路方式

* 基于单元格的导航图
* 基于可视点的导航图
* 导航网络

## 估价算法

### 曼哈顿距离(Manhattan distance)

* 不考虑斜向移动的情况，直线运动的代价为 **P/单位**

$$h(n)=P × (ABS(n.x-goal.x) + ABS(n.y-goal.y))$$


### 切比雪夫距离(Chebyshev distance)

* 对于对角线和直线运动的代价都为 **P/单位** 的时候（类似西洋棋中的王）

$$h(n)=P × MAX(ABS(n.x-goal.x),ABS(n.y-goal.y))$$

* 对角线(diagonal)运动在实际生活中运动的代价跟直线(straight)运动的代价(P/单位)并不同，对角线运动的代价一般类似于 $P_0=\sqrt{2}P$

$$h(n)=P_0 × h_{diagonal}(n)+P × (h_{straight}(n) - 2 × h_{diagonal}(n))$$

## 欧几里得距离(Euclidean distance)

* 欧氏距离就是两点之间直线距离

$$h(n)=P × \sqrt{(n.x-goal.x)^2 + (n.y-goal.y)^2}$$
- 注意：计算平方根将会消耗资源比较多，所以此方法基本不被使用


## 参考资料
https://www.redblobgames.com/pathfinding/a-star/introduction.html