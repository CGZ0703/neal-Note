# Evaluating Machine Learning Models 1

评价指标是机器学习任务中非常重要的一环。不同机器学习任务有不同的评价指标，同一种机器学习任务也有不同指标。每个指标的着重点不一样。

## 分类评价指标

### 混淆矩阵 Confusion Matrix

分类准确度在评价分类算法时，会有很大的问题的。分类算法的评价要比回归算法多很多。

对于一个癌症预测系统，输入检查指标，判断是否患有癌症，预测准确度99.9%。这个系统是好是坏呢？

如果癌症产生的概率是0.1%，那其实根本不需要任何机器学习算法，只要系统预测所有人都是健康的，即可达到99.9%的准确率。也就是说对于极度偏斜(Skewed Data)的数据，只使用分类准确度是不能衡量。

这是就需要使用混淆矩阵(Confusion Matrix)做进一步分析。

**混淆矩阵**
|  | Predicted as Positive | Predicted as Negative |
| --- | --- | --- |
| Labeled as Positive | True Positive(TP) | False Negative(FN) |
| Labeled as Negative | False Positive(FP) | True Negative(TN) |
> - 真正(True Positive, TP)：被模型分类正确的正样本。
> - 假负(False Negative, FN)：被模型分类错误的正样本。
> - 假正(False Positive, FP)：被模型分类的负样本。
> - 真负(True Negative, TN)：被模型分类正确的负样本。

由混淆矩阵可以计算这些评价指标： 准确率，平均准确率，精确率-召回率，F1-Score，ROC曲线

#### 准确率 Accuracy
$$
accuracy = \frac{TP + TN}{TP + FN + FP + TN}
$$

#### 平均准确率 Average Per-class Accuracy
$$
average\_accuracy = \frac{\frac{TP}{TP + FN} + \frac{TN}{TN + FP}}{2}
$$

#### 对数损失函数 Log-loss
$$
log_loss = -\frac{1}{N}\sum_{i=1}^N y_i log p_i + (1 - y_i)log(1 - p_i)
$$

#### 精确率-召回率 Precision-Recall
$$
Precision = \frac{TP}{TP + FP}
$$
$$
Recall = \frac{TP}{TP + FN}
$$


##### F1-score
$$
F1 = \frac{2 * precision * recall}{precision + recall}
$$

##### AUC - Area under the Curve (ROC - Receiver Operating Characteristic)



因为混淆矩阵表达的信息比简单的分类准确度更全面，因此可以通过混淆矩阵得到一些有效的指标。


## 回归评价指标

简单线性回归的目标：

已知训练数据样本`x,y`，找到a和b的值，使
$$
\sum_{i=1}^m (y^{(i)} - ax^i - b)^2
$$
尽可能小

实际上是找到训练数据集中的
$$
\sum (y_{train}^{(i)} - \widehat{y}\_train^{(i)})^2
$$
最小值

可以用
$$
\sum_{i=1}^m ( y_test^{(i)} - \widehat{y}_{trest}^{(i)})^2
$$
来作为衡量回归算法好坏的标准

### MSE (Mean Squared Error，均方误差)
测试集中的数据量m不同，因为有累加操作，所以随着数据的增加 ，误差会逐渐积累；因此衡量标准和 m 相关。为了抵消掉数据量的形象，可以除去数据量，抵消误差。
$$
\frac{1}{m} \sum_{i=1}^m (y_test^{(i)} - \widehat{y}_{trest}^{(i)})^2
$$

### RMSE (root mean square error，平方根误差)
使用均方误差MSE收到量纲的影响。例如在衡量房产时，y的单位是（万元），那么衡量标准得到的结果是（万元平方）。为了解决量纲的问题，可以将其开方（为了解决方差的量纲问题，将其开方得到平方差）得到均方根误差
$$
\sqrt{MSE_test} = \sqrt{\frac{1}{m}\sum_{i=1}^m (y_{test}^{(i)} - \widehat{y}_{trest}^{(i)})^2}
$$

### MAE (Mean Absolute Error，平均绝对误差)
对于线性回归算法还有另外一种非常朴素评测标准。要求真实值与预测结果之间的距离最小，可以直接相减做绝对值，加m次再除以m，即可求出平均距离，被称作平均绝对误差
$$
\frac{1}{m} \sum_{i=1}^m | y_{test}^{(i)} - \widehat{y}_{trest}^{(i)} |
$$

### Quantiles of Errors

### "Almost Crrect" Predictions

### R Squared
分类准确率，就是在01之间取值。但RMSE和MAE没有这样的性质，得到的误差。因此RMSE和MAE就有这样的局限性，比如我们在预测波士顿方差，RMSE值是4.9（万美元） 我们再去预测身高，可能得到的误差是10（厘米），我们不能说后者比前者更准确，因为二者的量纲根本就不是一类东西。

其实这种局限性，可以被解决。用一个新的指标R Squared。

$$
R^2 = 1 - \frac{SS_{residual}}{SS_{toal}} = 1 - \frac{\sum(\widehat{y}^{(i)} - y^{(i)})^2}{\sum(\overline{y} - y^{(i)})^2}
$$

**优点**：
- 对于分子来说，预测值和真实值之差的平方和，即使用我们的模型预测产生的错误。
- 对于分母来说，是均值和真实值之差的平方和，即认为“预测值=样本均值”这个模型（Baseline Model）所产生的错误。
- 我们使用Baseline模型产生的错误较多，我们使用自己的模型错误较少。因此用1减去较少的错误除以较多的错误，实际上是衡量了我们的模型拟合住数据的地方，即没有产生错误的相应指标。

**结论**：
- R^2 <= 1
- R^2越大也好，越大说明减数的分子小，错误率低；当我们预测模型不犯任何错误时，R^2最大值1
- 当我们的模型等于基准模型时，R^2 = 0
- 如果R^2 < 0，说明我们学习到的模型还不如基准模型。此时，很有可能我们的数据不存在任何线性关系。

