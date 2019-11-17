# Evaluating Machine Learning Models

在前面的kNN中，介绍了一个归类的模型，训练了一个模型后，如何对模型的效果，准确度做个描述？

能否直接拿到生产环境使用？

实际上，模型从建立好到真实使用，还差很多。

可以将原始数据中的一部分作为训练数据，另一部分作为测试数据。使用训练数据训练模型，再用测试数据看好坏。即通过测试数据判断模型好坏，然后再不断对模型进行修改。


## 使用以下指标可以评价模型

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

#### F1-score
$$
F1 = \frac{2 * precision * recall}{precision + recall}
$$

#### AUC - Area under the Curve (ROC - Receiver Operating Characteristic)


#### 混淆矩阵 Confusion Matrix

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

因为混淆矩阵表达的信息比简单的分类准确度更全面，因此可以通过混淆矩阵得到一些有效的指标。


#### 回归评价指标
  - RMSE (root mean square error，平方根误差)
  - Quantiles of Errors
  - “Almost Crrect” Predictions


















#### 排序评价指标

