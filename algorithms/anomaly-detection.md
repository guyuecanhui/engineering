# 异常检测

异常检测 (anomaly detection)，或者又被称为“离群点检测” (outlier detection)，是机器学习研究领域中跟现实紧密联系、有广泛应用需求的一类问题。但是，什么是异常，并没有标准答案，通常因具体应用场景而异。如果要给一个比较通用的定义，很多文献通常会引用 Hawkins 在文章开头那段话。很多后来者的说法，跟这个定义大同小异。这些定义虽然笼统，但其实暗含了认定“异常”的两个标准或者说假设：

* 异常数据跟样本中大多数数据不太一样。
* 异常数据在整体数据样本中占比比较小。

为了刻画异常数据的“不一样”，最直接的做法是利用各种统计的、距离的、密度的量化指标去描述数据样本跟其他样本的疏离程度。

---

<!--sec data-title="Isolation Forest" data-id="anomaly_0" data-show=true ces-->
### 主要思想
对一组数据进行随机切分（在某个特征的取值范围内随机选择一个值作为切分点），离群的点只需要较少次数的切分就能切分出来。根据这种思想，可以采用二叉树去对数据进行切分，数据点在二叉树中所处的深度反应了该条数据的“疏离”程度。

### 算法描述
算法分为训练和预测两部分：

**训练**：即抽取多个样本，构建多棵二叉树（Isolation Tree，即 iTree）。
```
从全量数据中抽取一批样本，用于构建一棵iTree;
随机选择一个特征作为起始节点，并在该特征的最大值和最小值之间随机选择一个值，将样本中小于该取值的数据划到左分支，大于等于该取值的划到右分支;
在左右两个分支数据中，重复步骤2，直到 a) 数据不可再分，即只包含一条数据，或者全部数据相同; 或 b) 二叉树达到限定的最大深度;
重复步骤1-3直到达到预设的iTree个数;
```

**预测**：综合多棵iTree的结果，计算每个数据点x的异常分值。对于某个iTree，假设x所在的叶节点有n条数据：
$$
H(x)=e+C(n)
$$
其中，$$e$$ 为x从iTree的根节点到叶节点过程中经过的边的数目，$$C(n)$$ 可以认为是一个修正值，它表示在一棵用n条样本数据构建的二叉树的平均路径长度。
$$
C(n)=2H(n-1)-\frac{2(n-1)}{n}
$$
其中，$$H(n-1) \approx \ln(n-1)+0.5772156649$$。
$$
S(x)=2^{-\frac{E(h(x))}{C(\psi)}}
$$
其中，$$E(h(x))$$ 表示数据 x 在多棵 iTree 的路径长度的均值，$$\psi$$ 表示单棵 iTree 的训练样本的样本数，$$C(\psi)$$ 表示用 $$\psi$$ 条数据构建的二叉树的平均路径长度，它在这里主要用来做归一化。 
```
计算x在每棵iTree上的路径长度H(x);
计算x的最终异常分值S(x);
```
从异常分值的公式看，如果数据 x 在多棵 iTree 中的平均路径长度越短，得分越接近 1，表明数据 x 越异常；如果数据 x 在多棵 iTree 中的平均路径长度越长，得分越接近 0，表示数据 x 越正常；如果数据 x 在多棵 iTree 中的平均路径长度接近整体均值，则打分会在 0.5 附近。 

### 算法应用
Isolation Forest 算法主要有两个参数：一个是二叉树的个数；另一个是训练单棵 iTree 时候抽取样本的数目。实验表明，当设定为 100 棵树，抽样样本数为 256 条时候，IF 在大多数情况下就已经可以取得不错的效果。这也体现了算法的简单、高效。

Isolation Forest 是无监督的异常检测算法，在实际应用时，并不需要黑白标签。需要注意的是：

1. 如果训练样本中异常样本的比例比较高，违背了先前提到的异常检测的基本假设，可能最终的效果会受影响；
2. 异常检测跟具体的应用场景紧密相关，算法检测出的“异常”不一定是我们实际想要的。比如，在识别虚假交易时，异常的交易未必就是虚假的交易。所以，在特征选择时，可能需要过滤不太相关的特征，以免识别出一些不太相关的“异常”。 

<!--endsec-->