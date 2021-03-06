# 蓄水池抽样 Reservoir Sampling

作为数据从业者，数据的采样必不可少。

给定一个数据流，数据流长度N很大，且N直到处理完所有数据之前都不可知，请问如何在只遍历一遍数据（O(N)）的情况下，能够随机选取出k个不重复的数据。

1. 数据流长度N很大且不可知，所以不能直接取N内的k个随机数，然后按索引取出数据。
2. 时间复杂度为O(N)，所以不能先遍历一遍，然后分块存储数据，再随机选取。
3. 随机选取k个数，每个数被选中的概率为k/N，必须保持数据选取绝对随机。

## 算法思路

### 可否在一未知大小的集合中，随机取出一元素？

例如在一很大，但未知确实行数的文字档中抽取任意一行。如果要确保每一行抽取的几率相等，即是说如果最后发现文字档共有N行，则每一行被抽取的几率均为1/N

第n行被抽取的几率为1/n，以Pn表示。如果档案共有N行，任意第n行被抽取的几率为

$$
\begin{aligned}
&P_n \prod_{k=n+1}^N (1-P_k)\\
=&\frac{1}{n} \prod_{k=n+1}^N (1-\frac{1}{k})\\
=&\frac{1}{n} \prod_{k=n+1}^N \frac{k-1}{k}\\
=&\frac{1}{n} \frac{n}{n+1} \frac{n+1}{n+2} ... \frac{N-1}{N}\\
=&\frac{1}{N}
\end{aligned}
$$

因此，各行被抽取的几率均相同。

因为这个算法是动态的概率，概率不仅跟N有关，每次加载一行都有可能会影响已产生的结果。所以保证了概率均等。

### 可否在一未知大小的集合中，随机取出k个元素？

也就是说，如果档案有 N >= k 行，则每一行被抽取的几率为 k/N

第n行被抽取的几率为k/n，以Pn表示。如果档案共有N行，任意第n行被抽取的几率为

$$
\begin{aligned}
&P_n \prod_{j=n+1}^N (1-\frac{P_j}{k})\\
=&\frac{k}{n} \prod_{j=n+1}^N (1-\frac{k}{kj})\\
=&\frac{k}{n} \prod_{j=n+1}^N \frac{j-1}{j}\\
=&\frac{k}{n} \frac{n}{n+1} \frac{n+1}{n+2} ... \frac{N-1}{N}\\
=&\frac{k}{N}
\end{aligned}
$$

伪代码：
```
从S中抽取首k项放入「蓄水池」中
对于每一个S[i](i>=k)：
    随机产生一个范围[0,i]的整数d
    若 d < k 则把蓄水池的第d项换成S[i]
```

## 代码实现

```python
import random


def select_k_iteams(stream, n, k):
    i = 0
    reservoir = [0] * k
    while i < k and i < n:
        reservoir[i] = stream[i]
        i += 1

    while i < n:
        d = random.randrange(i + 1)
        if d < k:
            reservoir[d] = stream[i]
        i += 1

    print("Following are k randomly selected stream items")
    print(reservoir)
```

## 分布式蓄水池抽样 Distributed/Parallel Reservoir Sampling

如果遇到超大的数据量，即使是O(N)的时间复杂度，蓄水池抽样程序完成抽样任务也将耗时很久。因此分布式的蓄水池抽样算法应运而生。

运作原理如下：

1. 假设有K台机器，将大数据集分成K个数据流
2. 每台机器使用单机版蓄水池抽样处理一个数据流，抽样m个数据，并最后记录处理的数据量为N1, N2, ..., Nk, ..., NK(假设m < Nk)。其中N1+N2+...+NK=N。
3. 取[1, N]一个随机数d，若d < N1，则在第一台机器的蓄水池中等概率不放回地（1/m）选取一个数据；若N1 <= d < (N1+N2)，则在第二台机器的蓄水池中等概率不放回地选取一个数据；一次类推，重复m次，则最终从N大数据集中选出m个数据。

m/N的概率验证如下：

1. 第k台机器中的蓄水池数据被选取的概率为m/Nk。
2. 从第k台机器的蓄水池中选取一个数据放进最终蓄水池的概率为Nk/N。
3. 第k台机器蓄水池的一个数据被选中的概率为1/m。（不放回选取时等概率的）
4. 重复m次选取，则每个数据被选中的概率为m \* (m / Nk \* Nk / N \* 1 / m)=m / N。
