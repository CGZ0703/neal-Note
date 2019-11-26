# 特征值归一化和标准化

前面有说过在实际应用中，样本的不同特征的数值范围不同，会在计算是产生比较大的影响，造成结果的不平衡。

在量纲不通过的情况下，未处理过的特征数值不能反映每一个特征的重要程度，这就需要做数据的归一/标准化了。

归一化就是将数据按比例缩放，使之落入一个小的特定区间。在某些比较和评价的指标处理中经常会用到，去除数据的单位限制，将其转化为无量纲的纯数值，便于不同单位或量级的指标能够进行比较和加权。

### 最值归一化(normalization)

把所有数据映射到0-1之间。最值归一化的使用范围是特征的分布具有明显边界的(分数0～100分、灰度0～255)，受outlier(异常点)的影响比较大

$$
X_{scale} = \frac{x - x_{min}}{x_{max} - x_{min}}
$$

如果需要将数据映射到 `[-1,1]`

$$
X_{scale} = \frac{x - x_{min}}{x_{max} - x_{min}} × 2 - 1
$$

也可以归一化到 固定区间 `[x, y]`

### 均值方差归一化(standardization)

把所有数据归一到均值为0方差为1的分布中。适用于数据中没有明显的边界，有可能存在极端数据值的情况.

$$
X_{scale} = \frac{x - x_{mean}}{S}
$$

证明： `E(X*)=0`，`D(X*)=1`

$$
E(X*) = E [\frac{X-E(X)}{\sqrt{D(X)}}] = \frac{1}{\sqrt{D(X)}E[X-E(X)]}=0
$$

$$
D(X*) = D [\frac{X-E(X)}{\sqrt{D(X)}}] = \frac{1}{\sqrt{D(X)}D[X-E(X)]} = \frac{D(X)}{D(X)} = 1
$$

Xmean为平均值，S是数据的标准差。

## sklearn的数据归一化

```python
import numpy as np
from sklearn import datasets

# 加载波斯顿房价数据
boston = datasets.load_boston()

# 提取数据集中的特征数据
X = boston.data
y = boston.target

# 在数据归一化之前：
# 最小值：0.0
# 最大值：711.0
# 平均值：70.07396704469443
# 标准差：145.1555388220164
print('在数据归一化之前：')
print('最小值：', X.min())
print('最大值：', X.max())
print('平均值：', X.mean())
print('标准差：', np.std(X))

# 导入最值归一化的方法
from sklearn.preprocessing import MinMaxScaler

# 对象实例化
minMaxScaler = MinMaxScaler()

# 类似于模型的训练过程
minMaxScaler.fit(X)

# 使用 transform 实现数据归一化
X_scale = minMaxScaler.transform(X)

# 在最值归一化之后：
# 最小值：0.0
# 最大值：1.0
# 平均值：0.3862566314283195
# 标准差：0.3423324125646035
print('=====================================')
print('在最值归一化之后：')
print('最小值：', X_scale.min())
print('最大值：', X_scale.max())
print('平均值：', X_scale.mean())
print('标准差：', np.std(X_scale))

# 导入均值方差归一化的方法
from sklearn.preprocessing import StandardScaler

# 对象实例化
stardardScaler = StandardScaler()

# 类似于模型的训练过程
stardardScaler.fit(X)

# 使用 transform 实现均值方差归一化
X_scale = stardardScaler.transform(X)

# 在均值方差归一化之后：
# 最小值：-3.9071933049810337
# 最大值：9.933930601860268
# 平均值：-1.1147462804871136e-15
# 标准差：0.9999999999999994
print('=====================================')
print('在均值方差归一化之后：')
print('最小值：', X_scale.min())
print('最大值：', X_scale.max())
print('平均值：', X_scale.mean())
print('标准差：', np.std(X_scale))
```

通过最值归一化之后，数据的最小值为 0， 最大值为 1，与数据归一化之前相比，平均值和标准差都明显减小了。

通过均值方差归一化之后，数据的平均值非常接近于 0，标准差非常接近于 1，与数据归一化之前相比，最小值与最大值之间的差距也明显缩小了。


## 自己实现的均值方差归一化

```python
import numpy as np

class StandardScaler:

    def __init__(self):
        self.mean_ = None
        self.scale_ = None

    def fit(self, X):
        """根据训练数据集X获得数据的均值和方差"""
        assert X.ndim == 2, "The dimension of X must be 2"

        # 求出每个列的均值
        self.mean_ = np.array([np.mean(X[:,i] for i in range(X.shape[1]))])
        self.scale_ = np.array([np.std(X[:, i] for i in range(X.shape[1]))])

        return self

    def tranform(self, X):
        """将X根据StandardScaler进行均值方差归一化处理"""
        assert X.ndim == 2, "The dimension of X must be 2"
        assert self.mean_ is not None and self.scale_ is not None, \
        "must fit before transform"
        assert X.shape[1] == len(self.mean_), \
        "the feature number of X must be equal to mean_ and std_"
        # 创建一个空的浮点型矩阵，大小和X相同
        resX = np.empty(shape=X.shape, dtype=float)
        # 对于每一列（维度）都计算
        for col in range(X.shape[1]):
            resX[:,col] = (X[:,col] - self.mean_[col]) / self.scale_[col]
        return resX
```

## 总结

我们在建模时要将数据集划分为训练数据集&测试数据集。

训练数据集进行归一化处理，需要计算出训练数据集的均值mean_train和方差std_train。

问题是：我们在对测试数据集进行归一化时，要计算测试数据的均值和方差么？

答案是否定的。在对测试数据集进行归一化时，仍然要使用训练数据集的均值train_mean和方差std_train。这是因为测试数据是模拟的真实环境，真实环境中可能无法得到均值和方差，对数据进行归一化。只能够使用公式`(x_test - mean_train) / std_train`并且，数据归一化也是算法的一部分，针对后面所有的数据，也应该做同样的处理.

因此我们要保存训练数据集中得到的均值和方差。

在sklearn中专门的用来数据归一化的方法：MinMaxScaler,StandardScaler。

### 需要进行归一化的模型

线性回归、多项式回归、逻辑回归、k-NN、神经网络、聚类、支持

### 不需要归一化的模型

对于概率的取值，因为不关心变量的取值，而是关心变量的分布和条件概率，所以不需要进行数据归一化。

0/1取值的特征通常不需要归一化，归一化会破坏它的稀疏性。
