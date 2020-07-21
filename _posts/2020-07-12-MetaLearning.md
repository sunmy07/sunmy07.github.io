---
layout: post
title:  "元学习文章阅读（MetaLearning）"
date:   2020-07-13 14:35:19
categories: Reading
tags: ML

---

<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

# 目录

* [目录](#目录)
* [MAML](#MAML)
  * [在监督分类中的应用](#在监督分类中的应用)
  * [关于二重梯度的进一步解释](#关于二重梯度的进一步解释)
  * [FOMAML一阶近似简化](#FOMAML一阶近似简化)
  * [缺点](#缺点)
* [Reptile](#Reptile)
* [各类算法实现](#各类算法实现)
* [参考文献](#参考文献)

# MAML

> The key idea underlying our method is to train the model’s initial parameters such that the model has maximal performance on a new task after the parameters have been up-dated through one or more gradient steps computed with a small amount of data from that new task.

本文的设想是**训练一组初始化参数**，通过在初始参数的基础上进行一或多步的梯度调整，来达到**仅用少量数据就能快速适应新task**的目的。为了达到这一目的，训练模型需要最大化新task的loss function的参数敏感度（*maximizing the sensitivity of the loss functions of new tasks with respect to the parameters*），当敏感度提高时，极小的参数（参数量）变化也可以对模型带来较大的改进。本文提出的算法可以适用于多个领域，包括少样本的回归、图像分类，以及增强学习，并且使用更少的参数量达到了当时（2017年）最先进的专注于少样本分类领域的网络的准确率。

核心算法示意图如下

![](..\assets\img\postsimg\20200713\3.jpg)

如上图所示，作者便将目标设定为，通过梯度迭代，找到对于task敏感的参数 θ 。训练完成后的模型具有对新task的学习域分布最敏感的参数，因此可以在仅一或多次的梯度迭代中获得最符合新任务的 θ* ，达到较高的准确率。

## 在监督分类中的应用

假设这样一个监督分类场景，目的是训练一个数学模型 $M_{fine-tune} $ ，对未知标签的图片做分类，则两大步骤如下：

1. 利用 $C_1 \sim C_{10} $ 的数据集训练元模型 $M_{meta} $ 
2. 在 $P_1 \sim P_5$ 的数据集上精调（fine-tune）得到最终的模型 $M_{fine-tune} $ 。

MAML在监督分类中的算法伪代码如下：

![](..\assets\img\postsimg\20200713\6.jpg)

下面进行详细分析。该算法是 meta-train 阶段，目的是得到 $M_{meta} $；

**第一个Require**，反复随机抽取 $ \mathcal T$，得到一批（e.g. 1000个）task池，作为训练集 $ p(\mathcal T)$。假设一个 $ \mathcal T$ 包含5类，每类20个样本，随机选5样本作为support set（$D_i$），剩余15样本为query set（$D_i'$）。

> 训练样本就这么多，要组合形成那么多的task，岂不是不同task之间会存在样本的重复？或者某些task的query set会成为其他task的support set？没错！就是这样！我们要记住，MAML的目的，在于fast adaptation，即通过对大量task的学习，获得足够强的泛化能力，从而面对新的、从未见过的task时，通过fine-tune就可以快速拟合。task之间，只要存在一定的差异即可。

**第二个Require**，step size就是学习率，MAML是基于二重梯度（gradient by gradient），每次迭代有两次参数更新过程，所以有两个学习率可以调整。

**1**：随机初始化模型参数 $\theta$；

**2**：循环，对于每个epoch，进行若干batch；

**3**：随机选取若干个（比如4个） $ \mathcal T$  形成一个batch；

**4**：对于每个batch中的第 $i$ 个 $ \mathcal T$ ，进行**第一次**梯度更新*「inner-loop 内层循环」*。

**5**：选取 $ \mathcal T_i$ 中的 **support set**，共  $N\cdot K$个样本（5-way 5-shot=25个样本）

**6**：计算每个参数的梯度。原文写对每一个类下的 $K$ 个样本做计算。实际上参与计算的总计有 $N\cdot K$ 个样本。这里的loss计算方法，在回归问题中就是MSE；在分类问题中就是cross-entropy；

**7**：进行第一次梯度更新得到 $\theta'$，可以理解为对 $ \mathcal T_i$ 复制一个原模型 $f(\theta)$ 来更新参数；

**8**：挑出训练集中的 query set 数据用于后续二次梯度更新；

**9**：完成第一次梯度更新。

**10**：进行**第二次**梯度更新，此时计算出的梯度直接通过GD作用于原模型上，用于更新其参数。*「outer-loop 外层循环」*

大致与步骤 **7** 相同，但是不同点有三处：

- 隐含了**二重梯度**，需要计算 $\mathcal L_{T_i}f(\theta')$ 对 $\theta$ 的导数，而 $\mathcal L_{T_i}f(\theta')$ 是 $\theta'$ 的函数， $\theta'$ 又是 $\theta$ 的函数（见步骤**7**）；

- 不再分别利用每个 $ \mathcal T$ 的loss更新梯度，而是计算一个 batch 中模型 $L_{T_i}f(\theta')$ 的 loss 总和进行梯度下降；

- 参与计算的样本是task中的 **query set**（5way*15=75个样本），目的是增强模型在task上的泛化能力，避免过拟合support set。

**11**：结束在该batch中的训练，回到步骤**3**，继续采样下一个batch。

**总结**：MAML使用训练集优化内层循环，使用测试集优化模型，也就是外层循环。外层循环需要计算**二重梯度（gradient by gradient）**。

## 关于二重梯度的进一步解释

设初始化的参数为 $\theta$ ，每个任务的模型一开始的参数都是 $\theta$。

模型在 $\mathcal T_i$ 上经过训练后，参数就会变成 $\theta_i'$ ，用 $l_i(\theta_i')$ 表示模型在**每个任务 $\mathcal T_i$ 上的损失**。那么，对于这个Meta Learning而言，**整体的损失**函数应该是 $\theta$ 的函数：

$$
\mathcal L( \theta ) = \sum_{i=1}^N l_i(\theta_i')
$$

一旦我们可以计算 $\theta$ 的梯度，就可以直接更新 $\theta$ ：

$$
\theta' \leftarrow \theta-\eta\nabla_{\theta}\mathcal L(\theta)
$$

而所谓的假设即是：**每次训练只进行一次梯度下降**。这个假设听起来不可思议，但是却也有一定的道理，首先我们只是在Meta Learning的过程中只进行一次参数下降，而真正学习到了很好的 $\theta$ 之后自然可以进行多次梯度下降。只考虑一次梯度下将的原因有：

- Meta Learning会**快**很多；
- 如果能让模型**只经过一次**梯度下降就性能优秀，当然很好；
- Few-shot learning的数据有限，多次梯度下降很容易**过拟合**；
- 刚才说的可以在实际应用中**多次梯度下降**。

如果只经历了一次梯度下降，模型最后的参数就会变成：

$$
\theta' = \theta-\epsilon \nabla_{\theta}\mathcal l(\theta)
$$

对于每个不同任务，准确来说应该是：

$$
\theta_i' = \theta-\epsilon \nabla_{\theta}\mathcal l_i(\theta)
$$

下一步就是计算 $\theta$ 关于 $\mathcal L$ 的梯度。我们有：

$$
\nabla_\theta \mathcal L( \theta ) = \nabla_\theta \sum_{i=1}^N l_i(\theta_i') = \sum_{i=1}^N \nabla_\theta l_i(\theta_i')
$$

现在的问题是如何求 $\nabla_\theta l_i(\theta_i')$ ，略去下标 $i$，有：

$$
\begin{aligned}
\nabla_\theta l(\theta ') =
\left[   
    \begin{array}
    {}
    \partial l(\theta')/\partial \theta_1\\
	\partial l(\theta')/\partial \theta_2\\
	\vdots\\
	\partial l(\theta')/\partial \theta_n
	\end{array}
\right]
\end{aligned}
$$

注意 $l$ 是的 $\theta'$ 的函数，而 $\theta'$ 又和每一个 $\theta_i$ 有关，因此有：

$$
\frac {\partial l(\theta')}{\partial \theta_i} = \sum_j\frac{\partial l(\theta')}{\partial \theta_j'}\frac{\partial \theta_j'}{\partial \theta_i}
$$

也就是说，每一个 $\theta_i$ 通过影响不同的 $\theta_j '$，从而影响到了 $l$。$l$ 和 $\theta$ 的关系是很直接的，我们可以直接求$\frac{\partial l(\theta ')}{\partial \theta_j'}$ ，现在的问题是怎么求 $\frac{\partial \theta_j'}{\partial \theta_i}$。

注意到 $\theta'$ 和 $\theta$ 的关系也是显然的：

$$
\theta' = \theta-\epsilon \nabla_{\theta}\mathcal l(\theta)
$$

当 $i \neq j$ 时

$$
\frac{\partial \theta_j'}{\partial \theta_i} = 
- \epsilon\frac{\partial l^2(\theta)}{\partial \theta_i\partial \theta_j}
$$

当 $i = j$ 时

$$
\frac{\partial \theta_j'}{\partial \theta_i} = 
1 - \epsilon\frac{\partial l^2(\theta)}{\partial \theta_i\partial \theta_i}
$$

到此为止已经把梯度计算出来了，二重梯度也是MAML计算中最为耗时的部分。

## FOMAML一阶近似简化

在MAML的论文中提到了一种简化，它通过计算一重梯度来近似二重梯度。具体而言，假设学习率 $\epsilon \rightarrow 0^+$，则更新一次后的参数 $\theta'$ 对初始参数 $\theta$ 求偏导可变为

$$
\begin{align}
(i \neq j) \; \frac{\partial \theta_j'}{\partial \theta_i} &= 
- \epsilon\frac{\partial l^2(\theta)}{\partial \theta_i\partial \theta_j} \approx 0 \\
(i = j) \; \frac{\partial \theta_j'}{\partial \theta_i} &= 1 - \epsilon\frac{\partial l^2(\theta)}{\partial \theta_i\partial \theta_i} \approx 1
\end{align}
$$

那么原来的偏导可近似为：

$$
\frac {\partial l(\theta')}{\partial \theta_i} = \sum_j\frac{\partial l(\theta')}{\partial \theta_j'}\frac{\partial \theta_j'}{\partial \theta_i} \approx
\frac {\partial l(\theta')}{\partial \theta_i'}
$$

整个梯度就可以近似为：

$$
\nabla_\theta l(\theta') \approx \nabla_{\theta'} l(\theta')
$$

简化后的一阶近似的MAML模型参数更新式为：

$$
\theta \leftarrow \theta - \alpha \nabla_{\theta'} \sum \mathcal L(\theta')\\
\theta_i' \leftarrow \theta - \beta \nabla_\theta l_i(\theta)\\
$$

一阶近似的MAML可以看作是如下形式的参数更新：假设每个batch只有一个task，某次采用第m个task来更新模型参数，得到$\hat\theta^m$，再求一次梯度来更新模型的原始参数$\phi$，将其从 $\phi^0$ 更新至 $\phi^1$，以此类推。

![](..\assets\img\postsimg\20200713\4.jpg)

与之相比，右边是模型预训练方法，它是将参数根据每次的训练任务一阶导数的方向来更新参数。

## 缺点

MAML的缺点[[2](#ref2)]：

1. Hard to train：paper中给出的backbone是4层的conv加1层linear，试想，如果我们换成16层的VGG，每个task在算fast parameter的时候需要计算的Hessian矩阵将会变得非常大。那么你每一次迭代就需要很久，想要最后的model收敛就要更久。

2. Robustness一般：不是说MAML的robustness不好，因为也是由一阶online的优化方法SGD求解出来的，会相对找到一个flatten minima location。然而这和非gradient-based meta learning方法求解出来的model的robust肯定是没法比的。

# Reptile

Reptile是OpenAI提出的一种非常简单的meta learning 算法。与MAML类似也是学习网络参数的初始值。算法伪代码如下

![image-20200717113006074](..\assets\img\postsimg\20200713\7.jpg)

其中，$\phi$ 是模型的初始参数，$\tau$ 是某个 task，$SGD(L,\phi,k)$ 表示从$\phi$ 开始对损失函数$L$进行$k$次随机梯度下降，返回更新后的参数$W$。

在最后一步中，通过 $W-\phi$ 这种残差形式来更新一次初始参数。

算法当然也可设计为batch模式，如下

![image-20200717115435570](..\assets\img\postsimg\20200713\8.jpg)

如果k=1，该算法等价于「联合训练」（joint training，通过训练来最小化在一系列训练任务上期望损失）。

Reptile 要求 k>1，更新依赖于损失函数的高阶导数，k>1 时 Reptile 的行为与 k=1（联合训练）时截然不同。

Reptile与FOMAML紧密相关，但是与FOMAML不同，Reptile**无需对每一个任务进行训练-测试（training-testing）划分**。

相比MAML需要进行二重梯度计算，Reptile只需要进行一重梯度计算，计算速度更快。

## 分析

为什么 Reptile 有效？首先以两步 SGD 为例分析参数更新过程

$$
\begin{align}
\phi_0 &= \phi\\
\phi_1 &= \phi_0 - \alpha L_0'(\phi_0)\\
\phi_2 &= \phi_1 - \alpha L_1'(\phi_1) \\
& = \phi_0 - \alpha L_0'(\phi_0) - \alpha L_1'(\phi_1)
\end{align}
$$

下面定义几个**辅助变量**

$$
\begin{align}
g_i &= L_i'(\phi_i)
\;\;(gradient \; obtained\; during\;SGD)\\
\phi_{i+1} &= \phi_i-\alpha g_i
\;\;(sequence\;of\;parameters)\\
\overline{g}_i &= L_i'(\phi_0)
\;\;(gradient\;at\;initial\;point)\\
\overline{H}_i &= L_i''(\phi_0)
\;\;(Hessian\;at\;initial\;point)\\
\end{align}
$$

采用泰勒展开对 $g_i$ 展开至二阶导加高次项的形式

$$
\begin{align}
g_i = L_i'(\phi_i) &= L_i'(\phi_0) + L_i''(\phi_0)(\phi_i - \phi_0) + O(||\phi_i - \phi_0||^2)\\
&= \overline{g}_i + \overline{H}_i(\phi_i - \phi_0) + O(\alpha^2)\\
&= \overline{g}_i - \alpha\overline{H}_i\sum_0^{i-1}g_i + O(\alpha^2)\\
&= \overline{g}_i - \alpha\overline{H}_i\sum_0^{i-1}\overline{g}_i + O(\alpha^2)\\
\end{align}
$$

最后一步的依据是，$g_i = \overline{g}_i + O(\alpha)$ 带入倒数第二行时，后面的 $O(\alpha)$ 与求和符号前的 $\alpha$ 相乘，即变为 $O(\alpha^2)$ 从而合并为一项。

下面取k=2，即两步，分别推导 MAML、FOMAML、Reptile 的梯度。

对于二阶的MAML，初始参数 $\phi_0$ 首先在support set上梯度更新一次得到 $\phi_1$ ，然后将 $\phi_1$ 在 query set 上计算损失函数，再计算梯度更新模型的初始参数。即 query set 的 loss 要对 $\phi_0$ 求导，链式法则 loss 对  $\phi_1$ 求导乘以  $\phi_1$ 对 $\phi_0$ 求导

$$
\begin{align}
g_{MAML} &= \frac{\partial}{\partial\phi_0}L_1(\phi_1) = \frac{\partial \phi_1}{\partial \phi_0} L_1'(\phi_1) \\
& = (I-\alpha L_0''(\phi_0))L_1'(\phi_1)\\
& = (I-\alpha L_0''(\phi_0))(L_1'(\phi_0) + L_1''(\phi_0)(\phi_1 - \phi_0) + O(\alpha^2))\\
& = (I-\alpha L_0''(\phi_0))(L_1'(\phi_0) + L_1''(\phi_0)(\phi_1 - \phi_0)) + O(\alpha^2)\\
& = (I-\alpha L_0''(\phi_0))(L_1'(\phi_0) - \alpha L_1''(\phi_0)L_0'(\phi_0)) + O(\alpha^2)\\
& = L_1'(\phi_0)-\alpha L_0''(\phi_0)L_1'(\phi_0) - \alpha L_1''(\phi_0)L_0'(\phi_0) + O(\alpha^2)\\
\end{align}
$$

FOMAML进行的一阶简化，是参数 $\phi_1$ 对初始参数 $\phi_0$ 求导的部分，即 $\frac{\partial \phi_i}{\partial \phi_0} = const$，参见2.3节。则只剩下 loss 对参数 $\phi_1$ 的求导。

$$
\begin{align}
g_{FOMAML} &= L_1'(\phi_i) = L_1'(\phi_0) + L_1''(\phi_0)(\phi_1 - \phi_0) + O(\alpha^2)\\
&= L_1'(\phi_0) -\alpha L_1''(\phi_0)L_0'(\phi_0) + O(\alpha^2)
\end{align}
$$

对于Reptile，根据梯度定义和SGD过程有
$$
\begin{align}
g_{Reptile} = (\phi_0 - \phi_2)/\alpha &= L_0'(\phi_0)+L_1'(\phi_1)\\
&= L_0'(\phi_0)+L_1'(\phi_0)-\alpha L_0''(\phi_1)L_0'(\phi_0) + O(\alpha^2)
\end{align}
$$
接下来对上面的三个梯度进行变量替换，全部用之前定义的辅助变量来表示
$$
\begin{align}
g_{MAML}& &= g_1 - \alpha \overline{H_0}\overline{g}_1-\alpha\overline{H_1}\overline{g}_0+O(\alpha^2)\\
g_{ROMAML} &= g_1 &= \overline{g}_1-\alpha\overline{H}_1\overline{g}_0+O(\alpha^2)\\
g_{Reptile} &= g_0+g_1 &= \overline{g}_0+\overline{g}_1-\alpha \overline{H}_1\overline{g}_0 + O(\alpha^2)\\
\end{align}
$$
再次定义两个期望。

第一个：AvgGrad 定义为期望loss的梯度
$$
AvgGrad = \mathbb E_{\tau,0}[\overline{g}_0]
$$
（-AvgGrad）是参数 $\phi$ 在最小化 joint training 问题中的下降方向，即为任务的期望损失。

第二个：AvgGradInner 定义为

# 各类算法实现

[dragen-1860 的 Pytorch 实现](https://github.com/dragen1860/MAML-Pytorch)：https://github.com/dragen1860/MAML-Pytorch

[Tensorflow实现](https://github.com/dragen1860/MAML-TensorFlow)：https://github.com/dragen1860/MAML-TensorFlow

[First-order近似实现Reptile](https://github.com/dragen1860/Reptile-Pytorch)：https://github.com/dragen1860/Reptile-Pytorch

# 参考文献

<span id="ref1">[1]</span>  [Rust-in](https://www.zhihu.com/people/rustinnnnn). [MAML 论文及代码阅读笔记](https://zhuanlan.zhihu.com/p/66926599).

<span id="ref2">[2]</span> 人工智障. [MAML算法，model-agnostic metalearnings?](https://www.zhihu.com/question/266497742/answer/550695031)

[3] [Veagau](https://www.cnblogs.com/veagau/). [【笔记】Reptile-一阶元学习算法](https://www.cnblogs.com/veagau/p/11816163.html)
