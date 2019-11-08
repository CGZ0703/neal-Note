# TF-IDF 特征加权

## 概述

`tf-idf`（英语：term frequency–inverse document frequency）是一种用作信息检索与文本挖掘的常用特征加权技术，同常用与于文本主题提取和分词加权等场景。

`tf-idf`是用以评估一字词对于一个文档集或一个语料库中的其中一份文档的重要程度的一种统计方法。字词的重要性随着它在文档中出现的次数成正比增加，但同时会随着它在语料库中出现的频率成反比下降。

`tf-idf`加权的各种形式常被搜寻引擎应用，作为文档与用户查询之间相关程度的度量或评级。除了`tf-idf`以外，网际网路上的搜寻引擎还会使用基于连结分析的评级方法，以确定文档在搜寻结果中出现的顺序。

`tf-idf`权重计算方法经常会和余弦相似性(cosine similarity)一同使用于向量空间模型中，用以判断两份文档之间的相似性。

## 原理

**词数**: 词数(term count)是指某一个词语$t_i$在该文档中出现的次数 $n_{i,j}$，但是考虑到文章有长短之分，为了方便不同文章的比较，防止结果偏向于长文档(同一词语在长文档中可能会比短文档有更高的词数，而不管这个词语是否是重要的)，引入了词频的概念。

**词频**: 在一份固定的文档中，词频(term frequency)指的是某一词语在该文档中出现的频率，这个数字对词数的标准化。对于某特定文档里的词语$t_i$来说，它的词频可以表示为：

$$
tf_{i,j} = \frac{n_{i,j}}{\sum_k{n_{k,j}}}
$$

以上式子中$n_{i,j}$是该词在文档$d_j$中的出现次数，而分母则是在文档$d_j$中所有字词$t_k$的出现次数之和。

**逆向档案频率**: 逆向档案频率(inverse document frequency)是一个词语普遍重要性的度量。这时需要一个预料库(corpus)，用于模拟语言的使用环境。某一特定词语的idf，可以由总文档数目除以包含该词语的文档的数目，再将得到的商除以10为底的对数得到：

$$
idf_i = lg\frac{|D|}{|\{j : t_i \in d_j\}|}
$$

- $|D|$：语料库的文档总数
- $|\{j : t_i \in d_j\}|$：包含词语$t_i$的文档数量（即$n_{i,j} \neq 0$的文档数量）
- 其中：如果一个词越常见，分母就越大，逆档案率就越小越接近0。如果词语不在预料库中，就导致分母为零，因此一般情况下会使用$1+|\{j : t_i \in d_j\}|$作为分母

IDF在应用中一般是采用业务相关语料离线计算。


**tf-idf**
然后结合`tf`和`idf`的值可以获得`tf-idf`：

$$
tfidf_{i,j} = tf_{i,j} \times idf_i ==> \frac{n_{i,j}}{\sum_k{n_{k,j}}} \times lg\frac{|D|}{|\{j : t_i \in d_j\}|}
$$

某一特定文档內的高词语频率，以及该词语在整个文档集合中的低文档频率，可以产生出高权重的`tf-idf`。因此，`tf-idf`倾向于过滤掉常见的词语，保留重要的词语。

## 算法代码

```python
import os
from math import log


def get_tf_idf(path, dic):
    tf_idf = {}
    fold_list = os.listdir(path)
    count = 0
    idf = {}
    for key in dic.keys():
        idf[key] = 1
        tf_idf[key] = 0
    for fold in fold_list:
        file_list = os.listdir(path + '/' + fold)
        count += len(file_list)
        for file in file_list:
            with open(path + '/' + fold + '/' + file) as f:
                text = f.read()
                for key in dic.keys():
                    if key in text:
                        idf[key] += 1
    for key, value in idf.items():
        idf[key] = log(count / value + 1)
    for key, value in tf_idf.items():
        tf_idf[key] = dic[key] * idf[key]
    return tf_idf
```

## 实例代码：

#### 结巴分词

```python
import jieba.analyse

str = "自然语言是人类智慧的结晶，自然语言处理是人工智能中最为困难的问题之一，而对自然语言处理的研究也是充满魅力和挑战的。"
tags = jieba.analyse.extract_tags(str, topK=10, withWeight=True, allowPOS=())

print(type(tags))
for tag in tags:
    print(tag)

'''
output:
<class 'list'>
('自然语言', 1.841460308682353)
('处理', 0.6365712538070588)
('人工智能', 0.5563544938523529)
('魅力', 0.46466251611823534)
('结晶', 0.4636520257341176)
('智慧', 0.43481814794117646)
('挑战', 0.38781205124764706)
('充满', 0.3740585030641177)
('最为', 0.35500622907588236)
('困难', 0.34422878637882354)
'''
```

越是重要的词语所给予的权重就越大。

#### Scikit-learn

Scikit-learn提供了自己训练的模型的接口

```python
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.feature_extraction.text import TfidfTransformer

x_train = ['这是 第一 篇 文章 ，',
           '这 篇 文章 是 第二 篇 文章 。',
           '这是 第三 篇 文章 。'
           ]
x_test = ['这是 第几 篇 文章 ？']
CV = CountVectorizer(max_features=10)
transformer = TfidfTransformer()
tf_idf = transformer.fit_transform(CV.fit_transform(x_train))
x_train_weight = tf_idf.toarray()
tf_idf = transformer.transform(CV.transform(x_test))
x_test_weight = tf_idf.toarray()
print(x_test_weight)

'''
output:
[[0.61335554 0.         0.         0.         0.78980693]]
'''
```

### 一般流程：

1. 首先用相关的文本训练IDF值保存文件；
2. 在项目中首先初始化文件到内存，保存为字典便于快速查询；
3. 新文档读入后实时计算TF值并查询相关IDF值计算出TF-IDF值。


## 总结

TF-IDF的优点是简单快速，而且容易理解。缺点是有时候用词频来衡量文章中的一个词的重要性不够全面，有时候重要的词出现的可能不够多，而且这种计算无法体现位置信息，无法体现词在上下文的重要性。如果要体现词的上下文结构，那么你可能需要使用word2vec算法来支持。

本质上IDF是一种试图抑制噪声的加权，单纯的以为文本频率小的单词就越重要，文本频率大的单词就越无用。这对于大部分文本信息，并不是完全正确的。IDF的简单结构并不能使提取的关键词，十分有效地反映单词的重要程度和特征词的分布情况，使其无法很好地完成对权值调整的功能。尤其是在同类语料库中，这一方法有很大弊端，往往一些同类文本的关键词被掩盖。例如：语料库D中教育类文章偏多，而文本d是一篇属于教育类的文章，那么教育类相关的词语的IDF值将会偏小，使提取文本关键词的召回率更低。因此才会有词语逆频率方式计算加权算法TF-IWF(Term Frequency-Inverse Word Frequency)。

并且，针对于短句应用TF-IDF提取关键词时，由于短句中每个词的IDF值往往是相同的，鉴于上述IDF天然的弱点，此算法应用于短句分析也显得不可靠，针对短句这种情况，可以结合词性和黑白名单以及搜索点击数据和业务打tag等，需要比较综合的方式来解决短句问题。


> 数据科学家联盟  
> 微信号 DataScientistClub  
> 一个有情怀的公众号，汇聚BAT和TMD的数据科学家，专注于大数据/数据仓库/数据分析/数据挖掘/深度学习/人工智能等数据科学领域！带着大家一起学习、一起成长、一起成为数据科学家！