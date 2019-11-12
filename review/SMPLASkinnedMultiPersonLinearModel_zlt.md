## SMPL: A Skinned Multi-Person Linear Model

#### Michael J. Black



## 1 Introduction

 a skinned vertex-based model  表示不同形体的自然的人体姿态

 a linear function of the elements of the pose rotation matrices

论文指出现在可行的方法不能很好地兼容图形软件和渲染引擎

SMPL的好处在于现实性并可以和现有图形软件兼容

Basic linear blend skinning(LBS)广泛使用但是关节连接处不太好 有taffy和bowtie的问题

![image-20191112210508996](C:\Users\zlt19\AppData\Roaming\Typora\typora-user-images\image-20191112210508996.png)

Two main variants of SMPL are explored,one using linear blend skinning(LBS), and the other with Dual-Quaternion blend skinning (DQBS). A vertex-based, skinned, model such as SMPL is actually more accurate than a deformation-based model like BlendSCAPE.

## 2 Model

模型有N=6890个点和K=23个关键点

![image-20191112212409850](C:\Users\zlt19\AppData\Roaming\Typora\typora-user-images\image-20191112212409850.png)

#### Blend skinning

每个顶点会受到多个骨骼的控制，模型需要这些控制有合理的权重

这一部分需要处理点的生成。

![image-20191112213256831](C:\Users\zlt19\AppData\Roaming\Typora\typora-user-images\image-20191112213256831.png)

然后还有偏移量要学习，加上偏移量就得到了最后的点坐标。

#### Shape Blend Shape

线性 的来源，是一组正交主成分的加权的结果。
$$
B_{S}(\vec{\beta} ; \mathcal{S})=\sum_{n=1}^{|\vec{\beta}|} \beta_{n} \mathbf{S}_{n}
$$

#### Pose Blend Shape

$$
B_{P}(\vec{\theta} ; \mathcal{P})=\sum_{n=1}^{9 K}\left(R_{n}(\vec{\theta})-R_{n}\left(\vec{\theta}^{*}\right)\right) \mathbf{P}_{n}
$$

下标代表的是矩阵的第n个元素，均为3*3，所以求和至9K，theta\* 代表的是初始，所以差值代表着变化。

#### Joints Location 

$$
J(\vec{\beta} ; \mathcal{J}, \overline{\mathbf{T}}, \mathcal{S})=\mathcal{J}\left(\overline{\mathbf{T}}+B_{S}(\vec{\beta} ; \mathcal{S})\right)
$$

#### Final 

![image-20191112220353142](C:\Users\zlt19\AppData\Roaming\Typora\typora-user-images\image-20191112220353142.png)

## 3 Training

3D Scan转化成同SMPL的拓扑结构，需要registration，得到了ground truth.使用multi-shape dataset训练了男女两种参数，最小化生成模型和gt顶点的欧式距离和。还定义了一些其他的误差项，也有正则化的式子，为了优化模型表现。

论文后面做了一系列的评估，得到了平均欧氏距离和模型生成时间等。

### 一些结论和想法

SMPL在高质量的数据下，对不同体型和动作等信息进行了学习，blend shape是关于旋转矩阵的线性函数，复杂度更小，可以生成很多姿态并且速度更快。

SMPL是有Python框架的 有时间可以尝试一下。

对于SMPL模型的改进还有新的版本。

问题：我们利用SMPL做模型生成，是相当于直接使用这个工具吗？还是对其原理还须进一步把握对其模型生成原理做一些工作？那可能会很难