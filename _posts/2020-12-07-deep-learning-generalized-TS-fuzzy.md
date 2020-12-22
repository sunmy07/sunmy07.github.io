---
title: 深度学习文章阅读（广义TS模糊系统）
date: 2020-12-08 10:48:19 +0800
categories: [Academic, Paper]
tags: [fuzzy]
math: true
---

本文介绍了 2001 年 Taniguchi 等提出的一种广义 TS 模糊系统的建模方法、规则规约和鲁棒补偿方法。

<!--more-->

---
- [1. 引言](#1-引言)
- [2. 传统 TS 模糊系统](#2-传统-ts-模糊系统)
- [3. 广义 TS 模糊系统](#3-广义-ts-模糊系统)
  - [3.1. 建模](#31-建模)
  - [3.2. 建模举例](#32-建模举例)
  - [3.3. 规则约减](#33-规则约减)
  - [3.4. 举例](#34-举例)
- [4. 参考文献](#4-参考文献)

> T. Taniguchi; K. Tanaka; H. Ohtake; H.O. Wang. **Model construction, rule reduction, and robust compensation for generalized form of Takagi-Sugeno fuzzy systems**. IEEE Transactions on Fuzzy Systems ( Volume: 9, Issue: 4, Aug 2001).

# 1. 引言

在线性矩阵不等式（linear matrix inequality, LMI）设计框架下，基于 TS 模糊模型的非线性控制得以广泛应用。一般分为三个阶段：

- 第一阶段：对非线性被控对象的模糊建模
  - 利用输入输出数据进行模糊模型辨识（Takagi and Sugeno, 1993 等）
  - **或** 基于分区非线性思想的模糊系统构建（模糊 IF-THEN 规则）
- 第二阶段：模糊控制规则推导，它反映了模糊模型的规则结构，它通过所谓的并行分布式补偿（PDC）实现
- 第三阶段：模糊控制器设计，即确定反馈增益。

> This paper presents a systematic procedure of fuzzy control system design that consists of fuzzy model construction, rule reduction, and robust compensation for nonlinear systems. 
 
本文提出了一种模糊控制系统设计的系统程序，该程序由模糊模型构建，规则约简和非线性系统的鲁棒补偿组成。

注意，在本篇文章之前，还有两篇关键文章作为前续研究基础：

- H. O. Wang, K. Tanaka, and M. Griffin, “**Parallel distributed compensation of nonlinear systems by Takagi-Sugeno fuzzy model**,” in Proceedings of 1995 IEEE International Conference on Fuzzy Systems. The International Joint Conference of the Fourth IEEE International Conference on Fuzzy Systems and The Second International Fuzzy Engineering Symposium, Yokohama, Japan, **1995**, vol. 2, pp. 531–538, doi: 10.1109/FUZZY.1995.409737.
  **首次将TS模糊系统用于非线性系统的近似**，给出了模糊系统 Lyapunov 稳定性的充分条件，研究了并行分布式补偿（PDC），给出了状态反馈控制律下的稳定性判别准则。
- D. Localmodel, “**Stability Analysis of Fuzzy Control Systems**,” IEEE TRANSACTIONS ON SYSTEMS, MAN, AND CYBERNETICS-PART B: CYBERNETICS, vol. 26, no. 1, p. 4, **1996**.
  提出了模糊状态反馈控制器，貌似放宽了稳定性的条件（还没细看）。



# 2. 传统 TS 模糊系统

形式如下

$$
\begin{aligned}
  &rule\ i\quad (i=1,2,\cdots,r):\\
  &{\rm IF}\ z_1(t)\ is\ M_{1i}\ and\ \cdots\ and\ z_p(t)\ is\ M_{pi}\\
  &{\rm THEN}\ \dot \boldsymbol x(t) = \boldsymbol A_i \boldsymbol x(t) + \boldsymbol B_i \boldsymbol u(t)
\end{aligned}
$$

其中

- $r$ 是规则个数
- $M_{ji}$ 是第 $j$ 个输入分量的模糊集
- $\boldsymbol x\in \mathbb R^n$ 是状态量，$\boldsymbol u\in \mathbb R^m$ 是输入量
- $\boldsymbol A_i\in \mathbb R^{n\times n}, \boldsymbol B_i\in \mathbb R^{n\times m}$ 是系数矩阵
- $z_1(t),\cdots,z_p(t)$ 是已知的前提变量，可能为可测量的状态量的函数、外部干扰，和/或时间，用 $\boldsymbol z(t)$ 来表示所有

给定一对 $[\boldsymbol x(t),\boldsymbol u(t),\boldsymbol z(t)]$，采用重心法（加权平均法）可以得到模糊系统的最终输出：

$$
\dot \boldsymbol x(t) = \sum_{i=1}^r h_i(\boldsymbol z(t))(\boldsymbol A_i \boldsymbol x(t) + \boldsymbol B_i \boldsymbol u(t))$$

其中

$$
\begin{aligned}
  h_i(\boldsymbol z(t)) &= \frac{\omega_i(\boldsymbol z(t))}{\sum_{i=1}^r\omega_i(\boldsymbol z(t))}\\
  \omega_i(z(t)) &= \prod_{j=1}^p M_{ji}(z_j(t))
\end{aligned}
$$

$h_i(\boldsymbol z(t))$ 是每条规则的归一化权重，$M_{ji}(z_j(t)$ 是第 $i$ 条规则中第 $j$ 个分量的模糊集 $M_{ji}$ 的隶属度值。

参考：单点模糊产生器、乘积推理机和中心平均解模糊器。

> Wang H., Tanaka K. and Griifn M., Parallel distributed conpensation of nonlinear systems by Takagi and Sugeno’s fuzzy model, 1995, in Porc. 4 th IEEE Int. Conf. Fuzzy syst., Yokohama, Japan, pp: 531-538.

定理：当模糊规则条数适当时，模糊系统可以以任意的精度逼近实际的任意线性或非线性系统。（万能逼近器？）

> MamdaniE.H.and Assilian 5., Applications of fuzzy algorithms for control of simple dynamic Plant, IEEE Proc. Part-D, 1974, vol. 121, no. 8, pp: 1585-1588

# 3. 广义 TS 模糊系统

**个人前言**：传统的 TS 模糊系统是有规则个数的概念的，但是下文作者提出的广义 TS 模糊系统不再强调规则的概念，而是直接对状态方程进行模糊近似。

## 3.1. 建模

考虑某一类**非线性**系统表示如下：

$$
\dot x_i(t) = \sum_{j=1}^n f_{ij}(\boldsymbol z(t))x_j(t) + \sum_{k=1}^m g_{ik}(\boldsymbol z(t))u_k(t)
$$

其中：

- $x_1(t)\cdots x_n(t)$ 是状态量，$u_1(t)\cdots u_m(t)$ 是输入量
- $z_1(t),\cdots,z_n(t)$ 是已知的变量，**可能为状态量的函数**、外部变量，和/或时间
- $f_{ij}(\boldsymbol z(t)), g_{ik}(\boldsymbol z(t))$ 是关于 $\boldsymbol z(t)$ 矩阵

注意，上式中的 $j$ 是遍历所有状态量，$j=1,2,\cdots,n$，$k$ 是遍历所有输入量，$k=1,2,\cdots,m$。上式可以看作描述系统的**非线性**状态方程。该方程刻画了每个状态量的一阶导与所有状态量和控制（输入）量的线性组合关系。非线性可以体现在：**$z_i(t)$ 可为 $x_i(t)$ 的非线性函数**。假设 $z_i(t) = sin(x_i(t))$ ，$f_{ij}(\boldsymbol z(t))=\boldsymbol z(t) = sin(\boldsymbol x(t))$，则原始状态方程是关于 $\boldsymbol x(t)$ 的非线性方程组。

定义如下的新变量（表示系数 $f_{ij},g_{ik}$ 的最大最小值）

$$
\begin{aligned}
  a_{ij1} &\equiv \mathop{\rm max}\limits_{\boldsymbol z(t)} f_{ij}(\boldsymbol z(t))\\
  a_{ij2} &\equiv \mathop{\rm min}\limits_{\boldsymbol z(t)} f_{ij}(\boldsymbol z(t))\\
  b_{ik1} &\equiv \mathop{\rm max}\limits_{\boldsymbol z(t)} g_{ik}(\boldsymbol z(t))\\
  b_{ik2} &\equiv \mathop{\rm min}\limits_{\boldsymbol z(t)} g_{ik}(\boldsymbol z(t))\\
\end{aligned}
$$

借助上述新定义的变量，可以将 $f_{ij}(\boldsymbol z(t)), g_{ik}(\boldsymbol z(t))$ 转化为用其最大最小值表达的形式（transforming into fuzzy model representation）：

$$
\begin{aligned}
  f_{ij}(\boldsymbol z(t)) &= h_{ij1}(\boldsymbol z(t))a_{ij1} + h_{ij2}(\boldsymbol z(t))a_{ij2}\\
  g_{ik}(\boldsymbol z(t)) &= v_{ik1}(\boldsymbol z(t))b_{ik1} + v_{ik2}(\boldsymbol z(t))b_{ik2}
\end{aligned}
$$

上式的权重参数满足

$$
\sum_{l=1}^2 h_{ijl}(\boldsymbol z(t)) = 1\quad \sum_{l=1}^2 v_{ikl}(\boldsymbol z(t)) = 1
$$

权重参数实质上就是隶属度函数的形式，可以定义如下

$$
\begin{aligned}
h_{ij1}(\boldsymbol z(t)) &= \frac{f_{ij}(\boldsymbol z(t))-a_{ij2}}{a_{ij1}-a_{ij2}}\\
h_{ij2}(\boldsymbol z(t)) &= \frac{a_{ij1} - f_{ij}(\boldsymbol z(t))}{a_{ij1}-a_{ij2}}\\
v_{ik1}(\boldsymbol z(t)) &= \frac{g_{ik}(\boldsymbol z(t))-b_{ij2}}{b_{ij1}-b_{ij2}}\\
v_{ik2}(\boldsymbol z(t)) &= \frac{b_{ik1} - g_{ik}(\boldsymbol z(t))}{b_{ik1}-b_{ik2}}\\
\end{aligned}
$$

最终可以将原始TS模糊系统表示为：

$$
\begin{aligned}
\dot x_i(t) &= \sum_{j=1}^n f_{ij}(\boldsymbol z(t))x_j(t) + \sum_{k=1}^m g_{ik}(\boldsymbol z(t))u_k(t)\\
&=\sum_{j=1}^n\sum_{l=1}^2 h_{ijl}(\boldsymbol z(t))a_{ijl}x_j(t) + \sum_{k=1}^m\sum_{l=1}^2 v_{ikl}(\boldsymbol z(t))b_{ikl}u_k(t)
\end{aligned}
$$

$i$ 是输入向量的维度（表示状态方程的每个状态量），$j$ **也是**输入向量的维度（表示每个状态量的一阶导与所有状态量的关系），$l$ 是取大取小值的维度。<font color=red>（没看懂可以参考后面的建模举例）</font>

将上述式子转为矩阵形式，如下

$$
\begin{aligned}
\dot \boldsymbol x(t) &=\sum_{i=1}^n\sum_{j=1}^n\sum_{l=1}^2 h_{ijl}(\boldsymbol z(t))a_{ijl} \boldsymbol U^A_{ij} \boldsymbol x(t) + \sum_{i=1}^n\sum_{k=1}^m\sum_{l=1}^2 v_{ikl}(\boldsymbol z(t))b_{ikl}\boldsymbol U^B_{ik}\boldsymbol u(t)\\
&=\sum_{i=1}^n\sum_{j=1}^n\sum_{l=1}^2 h_{ijl}(\boldsymbol z(t)) \boldsymbol A_{ijl} \boldsymbol x(t) + \sum_{i=1}^n\sum_{k=1}^m\sum_{l=1}^2 v_{ikl}(\boldsymbol z(t))\boldsymbol B_{ikl}\boldsymbol u(t)
\end{aligned}
$$

其中

$$
\begin{aligned}
\boldsymbol A_{ijl} &= \begin{bmatrix}
  0&\cdots&0&0&0&\cdots&0\\
  \vdots&\vdots&\vdots&\vdots&\vdots&\vdots&\vdots\\
  0&\cdots&0&0&0&\cdots&0\\
  0&\cdots&0&a_{ijl}&0&\cdots&0\\
  0&\cdots&0&0&0&\cdots&0\\
  \vdots&\vdots&\vdots&\vdots&\vdots&\vdots&\vdots\\
  0&\cdots&0&0&0&\cdots&0\\
\end{bmatrix}\\
\boldsymbol B_{ikl} &= \begin{bmatrix}
  0&\cdots&0&0&0&\cdots&0\\
  \vdots&\vdots&\vdots&\vdots&\vdots&\vdots&\vdots\\
  0&\cdots&0&0&0&\cdots&0\\
  0&\cdots&0&b_{ijl}&0&\cdots&0\\
  0&\cdots&0&0&0&\cdots&0\\
  \vdots&\vdots&\vdots&\vdots&\vdots&\vdots&\vdots\\
  0&\cdots&0&0&0&\cdots&0\\
\end{bmatrix}
\end{aligned}
$$

个人理解：上述两个矩阵就是为了前面的矩阵求和式而人工构造的，可以与后面的 $\boldsymbol x(t), \boldsymbol u(t)$ 做矩阵乘法，取到对应的元素乘积。也就是说，上述两个矩阵分别有 $n,m$ 个，且每个都是上面这种只有一个元素不为 0 的稀疏形式。

作者表明，$a_{ijl}, b_{ikl}$ 再规则约减中非常重要，上面矩阵和的式子在规则约减中十分方便。

## 3.2. 建模举例

考虑如下非线性系统

$$
\begin{aligned}
\left\{
\begin{array}{l}
\dot x_1(t) = x_2(t)\\
\dot x_2(t) = x_1(t){\rm cos}x_2(t)-x_3(t)\\
\dot x_3(t) = x_1(t)x_3(t) + (1+\varepsilon {\rm sin}x_3(t))u(t)
\end{array}
\right.
\end{aligned}
$$

状态量的取值范围为

$$
\begin{aligned}
  \underline d_1\leq x_1(t)\leq \overline d_1\\
  \underline d_2\leq x_2(t)\leq \overline d_2\\
  \underline d_3\leq x_3(t)\leq \overline d_3\\
\end{aligned}
$$

其中

$$
\begin{aligned}
\underline d_1 &= -5\\
\overline d_1 &= 5\\
\underline d_2 &= -\pi/2\\
\overline d_2 &= \pi/2\\
\underline d_3 &= -\pi\\
\overline d_3 &= \pi\\
\varepsilon &= 0.5\\
\end{aligned}
$$

根据前面的约定，重新整理状态方程如下

$$
\begin{aligned}
\left\{
\begin{array}{l}
\dot x_1(t) &= 0\cdot x_1(t)& + 1\cdot x_2(t)& + 0\cdot x_3(t)& + 0\cdot u(t)&\\
\dot x_2(t) &= {\rm cos}x_2(t)\cdot x_1(t)& + 0\cdot x_2(t)& +(-1) \cdot x_3(t)& + 0\cdot u(t)&\\
\dot x_3(t) &= 0\cdot x_1(t)& +0\cdot x_2(t)& + x_1(t)\cdot x_3(t)& + (1+\varepsilon {\rm sin}x_3(t))u(t)&
\end{array}
\right.
\end{aligned}
$$

有

$$
\begin{aligned}
f_{11}(z(t)) &= 0,\ &f_{12}(z(t)) = 1,\ &f_{13}(z(t)) = 0,\ &g_{11}(z(t)) = 0\\
f_{21}(z(t)) &= {\rm cos}x_2(t),\ &f_{22}(z(t)) = 0,\ &f_{23}(z(t)) = -1,\ &g_{11}(z(t)) = 0\\
f_{31}(z(t)) &= 0,\ &f_{32}(z(t)) = 0,\ &f_{33}(z(t)) = x_1(t),\ &g_{11}(z(t)) = (1+\varepsilon {\rm sin}x_3(t))\\
\end{aligned}
$$

计算系数的最大最小值，有（加 * 号的是原文中列出的，略去了  $a=b=0$ 项）

$$
\begin{aligned}
a_{111} &= 0,\ &a_{112} = 0\\
*a_{121} &= 1,\ &a_{122} = 1\\
a_{131} &= 0,\ &a_{132} = 0\\
b_{111} &= 0,\ &b_{112} = 0\\
\\
*a_{211} &= 1,\ &a_{212} = 0\\
a_{221} &= 0,\ &a_{222} = 0\\
*a_{231} &= -1,\ &a_{232} = -1\\
b_{211} &= 0,\ &b_{212} = 0\\
\\
a_{311} &= 0,\ &a_{312} = 0\\
a_{321} &= 0,\ &a_{322} = 0\\
*a_{331} &= 5,\ &a_{332} = -5\\
*b_{311} &= 1.5,\ &b_{312} = 0.5\\
\end{aligned}
$$

隶属度函数指定为（加 * 号的是原文中列出的，略去了 $a=b=0$ 对应项）

$$
\begin{aligned}
h_{111}(z(t)) &= 0.5,\ &h_{112}(z(t)) = 0.5\\
*h_{121}(z(t)) &= 0.5,\ &h_{122}(z(t)) = 0.5\\
h_{131}(z(t)) &= 0.5,\ &h_{132}(z(t)) = 0.5\\
v_{111}(z(t)) &= 0.5,\ &v_{112}(z(t)) = 0.5\\
\\
*h_{211}(z(t)) &= {\rm cos}x_2(t),\ &h_{212}(z(t)) = 1-{\rm cos}x_2(t)\\
h_{221}(z(t)) &= 0.5,\ &h_{222}(z(t)) = 0.5\\
*h_{231}(z(t)) &= 0.5,\ &h_{232}(z(t)) = 0.5\\
v_{211}(z(t)) &= 0.5,\ &v_{212}(z(t)) = 0.5\\
\\
h_{311}(z(t)) &= 0.5,\ &h_{312}(z(t)) = 0.5\\
h_{321}(z(t)) &= 0.5,\ &h_{322}(z(t)) = 0.5\\
*h_{331}(z(t)) &= \frac{x_1(t)+5}{10},\ &h_{332}(z(t)) = \frac{5-x_1(t)}{10}\\
*v_{311}(z(t)) &= \frac{1+{\rm sin}x_3(t)}{2},\ &v_{312}(z(t)) = \frac{1-{\rm sin}x_3(t)}{2}\\
\end{aligned}
$$

根据一般形式

$$
\dot \boldsymbol x(t) =\sum_{i=1}^n\sum_{j=1}^n\sum_{l=1}^2 h_{ijl}(\boldsymbol z(t))a_{ijl} \boldsymbol U^A_{ij} \boldsymbol x(t) + \sum_{i=1}^n\sum_{k=1}^m\sum_{l=1}^2 v_{ikl}(\boldsymbol z(t))b_{ikl}\boldsymbol U^B_{ik}\boldsymbol u(t)
$$

则原系统对应的模糊模型可以写为（所有 0 项乘积忽略）

$$
\begin{aligned}
\dot \boldsymbol x(t) = &h_{121}a_{121}\boldsymbol U_{12}^A \boldsymbol x(t)+h_{122}a_{122}\boldsymbol U_{12}^A \boldsymbol x(t)+\\
& h_{211}a_{211}\boldsymbol U^A_{21} \boldsymbol x(t)+h_{212}a_{212}\boldsymbol U^A_{21} \boldsymbol x(t)+\\
& h_{231}a_{231}\boldsymbol U^A_{23} \boldsymbol x(t)+h_{232}a_{232}\boldsymbol U^A_{23} \boldsymbol x(t)+\\
& h_{331}a_{331}\boldsymbol U^A_{33} \boldsymbol x(t)+h_{332}a_{332}\boldsymbol U^A_{33} \boldsymbol x(t)+\\
& v_{311}b_{311}\boldsymbol U^B_{31} \boldsymbol u(t)+v_{312}b_{312}\boldsymbol U^B_{31} \boldsymbol u(t)
\end{aligned}
$$

进行矩阵化，$h, a$ 的前两个下标以及 $U$ 的下标为行列号，$h, a$ 的第三个下标为最大最小值的加权和作为元素值，列写出子矩阵

$$
\begin{aligned}
\boldsymbol A_1 &= h_{121}a_{121}\boldsymbol U_{12}^A\\
\boldsymbol A_2 &= h_{122}a_{122}\boldsymbol U_{12}^A\\
\boldsymbol A_3 &= h_{211}a_{211}\boldsymbol U_{21}^A\\
\boldsymbol A_4 &= h_{212}a_{212}\boldsymbol U_{21}^A\\
\boldsymbol A_5 &= h_{231}a_{231}\boldsymbol U_{23}^A\\
\boldsymbol A_6 &= h_{232}a_{232}\boldsymbol U_{23}^A\\
\boldsymbol A_7 &= h_{331}a_{331}\boldsymbol U_{33}^A\\
\boldsymbol A_8 &= h_{332}a_{332}\boldsymbol U_{33}^A\\
\boldsymbol B_1 &= v_{311}a_{311}\boldsymbol U_{31}^B\\
\boldsymbol B_2 &= v_{312}a_{312}\boldsymbol U_{31}^B\\
\end{aligned}
$$

令

$$
\begin{aligned}
\boldsymbol A &= \boldsymbol A_1+\boldsymbol A_2+\cdots+\boldsymbol A_8\\
\boldsymbol B &= \boldsymbol B_1+\boldsymbol B_2
\end{aligned}
$$

有

$$
\begin{aligned}
\dot \boldsymbol x(t) &= \boldsymbol A\boldsymbol x(t) + \boldsymbol B\boldsymbol u(t)\\
\end{aligned}
$$

**但是这并不是广义模糊 TS 系统的形式**，下面进行转换。

注意到（存在两组最大最小值相等的情况）

$$
a_{121}=a_{122}=1,\ a_{231}=a_{232}=-1
$$

则

$$
\begin{aligned}
\boldsymbol A_1 &= \boldsymbol A_2 = 0.5\boldsymbol U_{12}^A\\
\boldsymbol A_5 &= \boldsymbol A_6 = -0.5\boldsymbol U_{23}^A\\
\boldsymbol A_{b1} &= \boldsymbol A_1 + \boldsymbol A_2 = \begin{bmatrix}
  0&1&0\\
  0&0&0\\
  0&0&0
\end{bmatrix}\\
\boldsymbol A_{b2} &= \boldsymbol A_5 + \boldsymbol A_6 = \begin{bmatrix}
  0&0&0\\
  0&0&-1\\
  0&0&0
\end{bmatrix}\\
\boldsymbol A_{b} &= \boldsymbol A_{b1} + \boldsymbol A_{b2} = \begin{bmatrix}
  0&1&0\\
  0&0&-1\\
  0&0&0
\end{bmatrix}
\end{aligned}
$$

分析其它子矩阵。将子矩阵进行变换，如下

$$
\begin{aligned}
\boldsymbol A_3 &= h_{211}a_{211}\boldsymbol U_{21}^A = h_{211}(v_{311}+v_{312})(h_{331}+h_{332})a_{211}\boldsymbol U_{21}^A\\
\boldsymbol A_4 &= h_{212}a_{211}\boldsymbol U_{21}^A = h_{212}(v_{311}+v_{312})(h_{331}+h_{332})a_{212}\boldsymbol U_{21}^A\\
\boldsymbol A_7 &= h_{331}a_{331}\boldsymbol U_{33}^A = h_{331}(v_{311}+v_{312})(h_{211}+h_{212}) a_{331}\boldsymbol U_{33}^A\\
\boldsymbol A_8 &= h_{332}a_{332}\boldsymbol U_{33}^A = h_{332}(v_{311}+v_{312})(h_{211}+h_{212})a_{332}\boldsymbol U_{33}^A\\
\boldsymbol B_1 &= v_{311}b_{311}\boldsymbol U_{31}^B= v_{311}(h_{211}+h_{212})(h_{331}+h_{332})b_{311}\boldsymbol U_{31}^B\\
\boldsymbol B_2 &= v_{312}b_{312}\boldsymbol U_{31}^B= v_{312}(h_{211}+h_{212})(h_{331}+h_{332})b_{312}\boldsymbol U_{31}^B\\
\end{aligned}
$$

类似的，将前面的两个矩阵的系数进行配合

$$
\begin{aligned}
  \boldsymbol A_{b} &= (h_{211}+h_{212})(v_{311}+v_{312})(h_{331}+h_{332})\boldsymbol A_{b}\\
\end{aligned}
$$

将系数展开后得到 **八** 组系数

$$
\begin{aligned}
h_1 &= h_{211}h_{331}v_{311}\\
h_2 &= h_{212}h_{331}v_{311}\\
h_3 &= h_{211}h_{332}v_{311}\\
h_4 &= h_{212}h_{332}v_{311}\\
h_5 &= h_{211}h_{331}v_{312}\\
h_6 &= h_{212}h_{331}v_{312}\\
h_7 &= h_{211}h_{332}v_{312}\\
h_8 &= h_{212}h_{332}v_{312}\\
\end{aligned}
$$

则

$$
\begin{aligned}
  \boldsymbol A_{b} &= (h_1+h_2+h_3+h_4+h_5+h_6+h_7+h_8)\boldsymbol A_{b}\\
  \boldsymbol A_3 &=(h_1+h_3+h_5+h_7)a_{211}\boldsymbol U_{21}^A\\
  \boldsymbol A_4 &=(h_2+h_4+h_6+h_8)a_{212}\boldsymbol U_{21}^A\\
  \boldsymbol A_7 &=(h_1+h_2+h_5+h_6)a_{331}\boldsymbol U_{33}^A\\
  \boldsymbol A_8 &=(h_3+h_4+h_7+h_8)a_{332}\boldsymbol U_{33}^A\\
  \boldsymbol B_1 &= (h_1+h_2+h_3+h_4)\boldsymbol b_{311}U_{31}^B\\
  \boldsymbol B_2 &= (h_5+h_6+h_7+h_8)\boldsymbol b_{312}U_{31}^B\\
\end{aligned}
$$

按照系数 $h$ 重新整理矩阵，有

$$
\begin{aligned}
\boldsymbol A = &h_1\boldsymbol A_1+h_2\boldsymbol A_2+\cdots+h_8\boldsymbol A_8 +\\
&h_1\boldsymbol B_1 + h_2\boldsymbol B_2 + \cdots + h_8\boldsymbol B_8
\end{aligned}
$$

其中

$$
\begin{aligned}
  \boldsymbol A_1 &= \boldsymbol A_5 = \boldsymbol A_b + a_{211}\boldsymbol U_{21}^A + a_{331}\boldsymbol U_{33}^A =
  \begin{bmatrix}
  0&1&0\\
  \boldsymbol 1&0&-1\\
  0&0&\boldsymbol 5
  \end{bmatrix}\\
  \boldsymbol A_2 &= \boldsymbol A_6  = \boldsymbol A_b + a_{212}\boldsymbol U_{21}^A + a_{331}\boldsymbol U_{33}^A =
  \begin{bmatrix}
  0&1&0\\
  \boldsymbol 0&0&-1\\
  0&0&\boldsymbol 5
  \end{bmatrix}\\
  \boldsymbol A_3 &= \boldsymbol A_7 = \boldsymbol A_b + a_{211}\boldsymbol U_{21}^A + a_{332}\boldsymbol U_{33}^A =
  \begin{bmatrix}
  0&1&0\\
  \boldsymbol 1&0&-1\\
  0&0&-\boldsymbol 5
  \end{bmatrix}\\
  \boldsymbol A_4 &= \boldsymbol A_8 = \boldsymbol A_b + a_{212}\boldsymbol U_{21}^A + a_{332}\boldsymbol U_{33}^A =
  \begin{bmatrix}
  0&1&0\\
  \boldsymbol 0&0&-1\\
  0&0&-\boldsymbol 5
  \end{bmatrix}\\
  \boldsymbol B_1 &= \boldsymbol B_2 = \boldsymbol B_3 = \boldsymbol B_4 = b_{311}U_{31}^B =
  \begin{bmatrix}
  0\\
  0\\
  1.5
  \end{bmatrix}\\
  \boldsymbol B_5 &= \boldsymbol B_6 = \boldsymbol B_7 = \boldsymbol B_8 = b_{312}U_{31}^B =
  \begin{bmatrix}
  0\\
  0\\
  0.5
  \end{bmatrix}\\
  \end{aligned}
$$

至此终于推得文中 「**易得**」 的系数，汗！

按照整理后的系数和矩阵重新写系统状态方程，有

$$
\dot \boldsymbol x(t) = \sum_{i=1}^8 h_i(\boldsymbol z(t))[\boldsymbol A_i\boldsymbol x(t) + \boldsymbol B_i\boldsymbol u(t)]
$$

>Note that this fuzzy model has nonlinear terms in $A(2,1),A(3,3)$, and $B(3,1)$, where denotes the (2, 1) element of $A$ matrix.

原文说模糊系统的 $\boldsymbol A$ 的 $(2,1),(3,3)$ 元素和矩阵 $\boldsymbol B$ 的 $(3,1)$ 元素是非线性项。**个人**觉得，单纯从系数矩阵而言 $\boldsymbol A(3,3)$ 并不是非线性项，但是对于整个系统而言的确是非线性的。

---

进一步分析。上面的例子中，状态量维度为 3 即状态方程个数为 3，控制量维度为 1。那么一共有 $方程个数3\times 状态量个数3=9$ 个系数项 $f$， $方程个数3\times 控制量维度1=3$ 个系数项 $v$。

系统中有 4 项非零 $h$ 和 1 项非零 $v$。4 项非零 $h$ 中有两项系数为常数，对应的最大最小值相同。因此，系统一共有 $2+1=3$ 项非常数项系数。这三个系数进行排列组合乘法，就得到了 $C_2^1C_2^1C_2^1=8$ 个权重系数 $h$，也就对应 **8** 条模糊规则。

如果系统的所有系数项都不为常数，是否意味着一共有 $\underbrace{C_2^1\cdots C_2^1}_{12}=4096$ 条规则！？作者在后面分析了一般系统状态方程和广义 TS 模型之间的等价性（也即主要确定规则的个数）。

## 3.3. 规则约减

规则约减与使用 LMI 进行控制器设计的计算工作量密切相关。

基本思路：将非线性项 $f_{ij}(\boldsymbol z(t)), g_{ik}(\boldsymbol z(t))$ 替换为常数项 $a_{i_0j_0},b_{i_0k_0}$，其中 $a_{i_0j_0}=(a_{ij1}+a_{ij2})/2,\ b_{i_0k_0} = (b_{ik1}+b_{ik2})/2$。

对于任意的 $i_0,j_0$，对 $f_{i_0j_0}(\boldsymbol z(t))$ 约减后的模型为

$$
\begin{aligned}
\dot \boldsymbol x(t) = &\mathop{\sum_{i=1}^n\sum_{j=1}^n\sum_{l=1}^2}\limits_{(i,j)\neq (i_0,j_0)} h_{ijl}(\boldsymbol z(t))a_{ijl} \boldsymbol U^A_{ij} \boldsymbol x(t)\\
&+a_{i_0j_0}\boldsymbol U^A_{i_0j_0} \boldsymbol x(t)\\
&+\sum_{i=1}^n\sum_{k=1}^m\sum_{l=1}^2 v_{ijl}(\boldsymbol z(t))b_{ikl}\boldsymbol U^B_{ik}\boldsymbol u(t)
\end{aligned}
$$

类似地，对于任意的 $i_0,k_0$，对 $g_{i_0j_0}(\boldsymbol z(t))$ 约减后的模型为

$$
\begin{aligned}
\dot \boldsymbol x(t) = &\sum_{i=1}^n\sum_{j=1}^n\sum_{l=1}^2 h_{ijl}(\boldsymbol z(t))a_{ijl} \boldsymbol U^A_{ij} \boldsymbol x(t)\\
&+\mathop{\sum_{i=1}^n\sum_{k=1}^m\sum_{l=1}^2}\limits_{(i,k)\neq (i_0,k_0)} v_{ikl}(\boldsymbol z(t))b_{ikl}\boldsymbol U^B_{ik}\boldsymbol u(t)\\
&+b_{i_0k_0}\boldsymbol U^B_{i_0k_0} \boldsymbol u(t)
\end{aligned}
$$

下面分析了一般系统状态方程和广义 TS 模型之间的**等价性**（也即主要确定规则的个数），首先给出结论

$$
\begin{aligned}
  \dot \boldsymbol x(t) &=\sum_{i=1}^n\sum_{j=1}^n\sum_{l=1}^2 h_{ijl}(\boldsymbol z(t))a_{ijl} \boldsymbol U^A_{ij} \boldsymbol x(t)+\sum_{i=1}^n\sum_{k=1}^m\sum_{l=1}^2 v_{ikl}(\boldsymbol z(t))b_{ikl}\boldsymbol U^B_{ik}\boldsymbol u(t)\\
  &=\sum_{p=1}^{2^{n(n+m)}} h_p(\boldsymbol z(t))[\boldsymbol A_p\boldsymbol x(t) + \boldsymbol B_p\boldsymbol u(t)]                                     
\end{aligned}
$$

下面进行一步步推导分析（原文中又是一个 where 易得，我人傻了）。

<!-- 原模型状态方程展开后形如

$$
\begin{aligned}
  \dot \boldsymbol x(t) =
  &(h_{111}a_{111}+h_{112}a_{112}) \boldsymbol U^A_{11} \boldsymbol x(t)\\
  &+(h_{121}a_{121}+h_{122}a_{122}) \boldsymbol U^A_{12} \boldsymbol x(t)\\
  &+\cdots\\
  &+(h_{1n1}a_{1n1}+h_{1n2}a_{1n2}) \boldsymbol U^A_{1n} \boldsymbol x(t)\quad <1,j,l>\\
  &\\
  &+(h_{211}a_{211}+h_{212}a_{212}) \boldsymbol U^A_{21} \boldsymbol x(t)\\
  &+(h_{221}a_{221}+h_{222}a_{222}) \boldsymbol U^A_{22} \boldsymbol x(t)\\
  &+\cdots\\
  &+(h_{2n1}a_{2n1}+h_{2n2}a_{2n2}) \boldsymbol U^A_{2n} \boldsymbol x(t)\quad <2,j,l>\\
  &+\cdots+<i,j,l>+\cdots<n,j,l>\\
  &+<1,k,l>+\cdots+<m,k,l>
\end{aligned}
$$ -->

利用各项系数和为 1 的性质，进行乘法展开

$$
\begin{aligned}
  1 = &\overbrace{(h_{111}+h_{112})\cdots (h_{1n1}+h_{1n2})}^{n}&<1>\\
  &\cdot(h_{211}+h_{212})\cdots (h_{2n1}+h_{2n2})&<2>\\
  &\cdot\quad \cdots&\cdots\\
  &\cdot(h_{n11}+h_{n12})\cdots (h_{nn1}+h_{nn2})&<n>\\
  &\cdot\overbrace{(v_{111}+v_{n12})\cdots (v_{1m1}+v_{nm2})}^{m}&<1>\\
  &\cdot\quad \cdots&\cdots\\
  &\cdot(v_{n11}+v_{n12})\cdots (v_{nm1}+v_{nm2})&<n>\\
\end{aligned}
$$

上式中一共有 $n\cdot n+n\cdot m$ 个括号，从每个括号中任意取一个元素（$l=1\ or\ 2$）组成连乘项。第一行展开后共有 $2^n$ 项，则前 $n$ 行一共有 $2^{n\cdot n}$ 项。类似地，后 $n$ 行中，第一行展开后共有 $2^m$ 项，则后 $n$ 行一共有 $2^{m\cdot n}$ 项。那么，整个式子一共有 $2^{n\cdot n}2^{m\cdot n}=2^{n(n+m)}$ 项。每一项都是所有 $i,j,k$ 对不同 $l$ 的排列组合，即一共有 $2^{n(n+m)}$ 种排列组合。

假设所有对 $l$ 的选取均为 $l=1$（上式选取所有括号左边的元素组成一项），则该项为

$$
\begin{aligned}
  t_{l=\underbrace{1\cdots1}_{n\cdot n}\underbrace{1\cdots1}_{m\cdot n}} = &(h_{111}\cdots h_{1n1})\cdots(h_{n11}\cdots h_{nn1})\\
  &\cdot(v_{111}\cdots v_{1n1})\cdots(v_{1m1}\cdots v_{nm1})\\
  = &\prod_{i=1}^n (h_{i11}\cdots h_{in1}) \cdot (v_{i11} \cdots v_{im1})\\
  = &\prod_{i=1}^n \prod_{j=1}^n h_{ij1}\cdot (v_{i11}\cdots v_{im1})\\
  = &\prod_{i=1}^n\prod_{j=1}^n\prod_{k=1}^m h_{ij1}v_{ik1}
\end{aligned}
$$

对所有连乘项进行一般化，并进行求和，得到原始乘法展开的简单写法

$$
1 = \sum_{p=1}^{2^{n(n+m)}} \prod_{i=1}^n\prod_{j=1}^n\prod_{k=1}^m h_{ijl}v_{ikl}
$$


其中 $l$ 与选择有关。

那么

$$
\begin{aligned}
  \dot \boldsymbol x(t) &=\sum_{i=1}^n\sum_{j=1}^n\sum_{l=1}^2 h_{ijl}a_{ijl}\boldsymbol U_{ij}^A + \sum_{i=1}^n\sum_{k=1}^m\sum_{l=1}^2 v_{ijl}(\boldsymbol z(t))b_{ikl}\boldsymbol U^B_{ik}\boldsymbol u(t)\\
  &= \sum_{i=1}^n\sum_{j=1}^n\sum_{l=1}^2h_{ijl}\boldsymbol A_{ijl} + \sum_{i=1}^n\sum_{k=1}^m\sum_{l=1}^2v_{ikl}\boldsymbol B_{ijl}\\
  &=\sum_{p=1}^{2^{n(n+m)}} \prod_{i=1}^n\prod_{j=1}^n\prod_{k=1}^m h_{ijl}v_{ikl} \left[ \sum_{i=1}^n\sum_{j=1}^n\sum_{l=1}^2h_{ijl}\boldsymbol A_{ijl} + \sum_{i=1}^n\sum_{k=1}^m\sum_{l=1}^2v_{ikl}\boldsymbol B_{ijl}\right] \\
\end{aligned}
$$

同理

$$
v_{ikl}b_{ikl} =\sum_{p=1}^{2^{n(n+m)}} v_{ikl}b_{ikl} \prod_{i=1}^n\prod_{j=1}^n\prod_{k=1}^m h_{ikl}v_{ikl}
$$

## 3.4. 举例



# 4. 参考文献

无。