# 机器学习基本概念

---

<!--sec data-title="激活函数" data-id="basics_0" data-show=true ces-->
激活函数的目的是为了调节权重和误差，为模型引入非线性因素。

### [常用激活函数](http://mp.weixin.qq.com/s/7DgiXCNBS5vb07WIKTFYRQ)
1. **Relu**：修正线性单元（Rectified linear unit，ReLU）是神经网络中最常用的激活函数。它保留了 step 函数的生物学启发（只有输入超出阈值时神经元才激活），不过当输入为正的时候，导数不为零，从而允许基于梯度的学习（尽管在 x=0 的时候，导数是未定义的）。使用这个函数能使计算变得很快，因为无论是函数还是其导数都不包含复杂的数学运算。然而，当输入为负值的时候，ReLU 的学习速度可能会变得很慢，甚至使神经元直接无效，因为此时输入小于零而梯度为零，从而其权重无法得到更新，在剩下的训练过程中会一直保持静默。 
2. **Sigmoid**：Sigmoid 因其在 logistic 回归中的重要地位而被人熟知，值域在 0 到 1 之间。Logistic Sigmoid（或者按通常的叫法，Sigmoid）激活函数给神经网络引进了概率的概念。它的导数是非零的，并且很容易计算（是其初始输出的函数）。然而，在分类任务中，sigmoid 正逐渐被 Tanh 函数取代作为标准的激活函数，因为后者为奇函数（关于原点对称）。
3. **Tanh**：在分类任务中，双曲正切函数（Tanh）逐渐取代 Sigmoid 函数作为标准的激活函数，其具有很多神经网络所钟爱的特征。它是完全可微分的，反对称，对称中心在原点。为了解决学习缓慢和/或梯度消失问题，可以使用这个函数的更加平缓的变体（log-log、softsign、symmetrical sigmoid 等等）
<!--endsec-->

<!--sec data-title="损失函数" data-id="basics_1" data-show=true ces-->
损失函数是用来评价模型的预测值 $$\hat{Y} =f(X)$$ 与真实值 $$Y$$ 的不一致程度，它是一个非负实值函数。通常使用 $$L(Y,f(x))$$ 来表示，损失函数越小，模型的性能就越好。    

设总有N个样本的样本集为 $$(X,Y)=(x_i,y_i),\ i\in [1,N]$$，其中 $$y_i$$ 为样本 $$i$$ 的真实值，$$\hat{y_i}=f(x_i),\ i\in [1,N]$$ 为样本 $$i$$ 的预测值，$$f$$ 为分类或者回归函数。那么总的损失函数为：
$$
L=\sum_{i=1}^N l(y_i,\hat{y_i})
$$

### 常用损失函数比较
常见的损失函数 $$l(y_i,\hat{y_i})$$ 有以下几种： 
1. **Zero-one Loss**：Zero-one Loss即0-1损失，它是一种较为简单的损失函数，如果预测值与目标值不相等，那么为1，否则为0，即：
$$
l(y_i,\hat{y_i})=\begin{cases}
1,\ y_i\neq \hat{y_i} \\
0,\ y_i=\hat{y_i}
\end{cases}
$$
可以看出上述的定义太过严格，如果真实值为1，预测值为0.999，那么预测应该正确，但是上述定义显然是判定为预测错误，那么可以进行改进为Perceptron Loss。 
2. **Perceptron Loss**：Perceptron Loss即为感知损失。即：
$$
l(y_i,\hat{y_i})=\begin{cases}
1,\ \mid y_i- \hat{y_i}\mid > t \\
0,\ \mid y_i- \hat{y_i}\mid \le t
\end{cases}
$$
其中，$$t$$ 是一个超参数阈值，如在PLA([Perceptron Learning Algorithm,感知机算法](http://kubicode.me/2015/08/06/Machine%20Learning/Perceptron-Learning-Algorithm/))中取 $$t=0.5$$。 
3. **[Hinge Loss](https://en.wikipedia.org/wiki/Hinge_loss)**：Hinge损失可以用来解决间隔最大化问题，如在SVM中解决几何间隔最大化问题，其定义如下：
$$
l(y_i,\hat{y_i})=\max(0,1-y_i\cdot \hat{y_i}),\ \ y_i\in \{-1,1\}
$$
4. **Log Loss**：在使用似然函数最大化时，其形式是进行连乘，但是为了便于处理，一般会套上log，这样便可以将连乘转化为求和，由于log函数是单调递增函数，因此不会改变优化结果。因此log类型的损失函数也是一种常见的损失函数，如在LR([Logistic Regression, 逻辑回归](chrome-extension://ikhdkkncnoglghljlkmcimlnlhkeamad/pdf-viewer/web/viewer.html?file=https%3A%2F%2Fpeople.eecs.berkeley.edu%2F~russell%2Fclasses%2Fcs194%2Ff11%2Flectures%2FCS194%2520Fall%25202011%2520Lecture%252006.pdf))中使用交叉熵(Cross Entropy)作为其损失函数。即：
$$
l(y_i,\hat{y_i})==y_i\cdot\log(\hat{y_i})+(1-y_i)\cdot\log(1-\hat{y_i}),\ \ y_i\in \{0,1\}
$$
规定 $$0\cdot \log(\cdot)=0$$。
5. **Square Loss**：Square Loss即平方误差，常用于回归中。即：
$$
l(y_i,\hat{y_i})==(y_i-\hat{y_i})^2,\ \ y_i,\hat{y_i}\in R
$$
6. **Absolute Loss**：Absolute Loss即绝对值误差，常用于回归中。即：
$$
l(y_i,\hat{y_i})==\mid y_i-\hat{y_i}\mid,\ \ y_i,\hat{y_i}\in R
$$
7. **Exponential Loss**：Exponential Loss为指数误差，常用于boosting算法中，如[AdaBoost](https://en.wikipedia.org/wiki/AdaBoost)。即：
$$
l(y_i,\hat{y_i})==exp(-y_i\cdot \hat{y_i}),\ \ y_i\in \{-1,1\}
$$
<!--endsec-->

<!--sec data-title="优化函数" data-id="basics_2" data-show=true ces-->
定义了损失函数后，可以用优化函数来更新参数，使得模型向着损失函数最小的目标不断更新并收敛。

### [常用激活函数](http://mp.weixin.qq.com/s/7DgiXCNBS5vb07WIKTFYRQ)
1. **Relu**：修正线性单元（Rectified linear unit，ReLU）是神经网络中最常用的激活函数。它保留了 step 函数的生物学启发（只有输入超出阈值时神经元才激活），不过当输入为正的时候，导数不为零，从而允许基于梯度的学习（尽管在 x=0 的时候，导数是未定义的）。使用这个函数能使计算变得很快，因为无论是函数还是其导数都不包含复杂的数学运算。然而，当输入为负值的时候，ReLU 的学习速度可能会变得很慢，甚至使神经元直接无效，因为此时输入小于零而梯度为零，从而其权重无法得到更新，在剩下的训练过程中会一直保持静默。 
2. **Sigmoid**：Sigmoid 因其在 logistic 回归中的重要地位而被人熟知，值域在 0 到 1 之间。Logistic Sigmoid（或者按通常的叫法，Sigmoid）激活函数给神经网络引进了概率的概念。它的导数是非零的，并且很容易计算（是其初始输出的函数）。然而，在分类任务中，sigmoid 正逐渐被 Tanh 函数取代作为标准的激活函数，因为后者为奇函数（关于原点对称）。
3. **Tanh**：在分类任务中，双曲正切函数（Tanh）逐渐取代 Sigmoid 函数作为标准的激活函数，其具有很多神经网络所钟爱的特征。它是完全可微分的，反对称，对称中心在原点。为了解决学习缓慢和/或梯度消失问题，可以使用这个函数的更加平缓的变体（log-log、softsign、symmetrical sigmoid 等等）
<!--endsec-->