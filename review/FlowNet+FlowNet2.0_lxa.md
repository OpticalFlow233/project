# FlowNet && FlowNet2.0

## The first case to utilize CNN to solve optical flow problem

## 1 Introduction

## 2 FlowNet

### 2.1 Structure of FlowNet

![image-20191105165824593](C:\Users\lxa\AppData\Roaming\Typora\typora-user-images\image-20191105165824593.png)

### 2.2 Introduction

- utilize 2 structures, generic CNN(feed data of image pairs(stack together) and GT, predict the x-y flow fields directly from the images, namely, FlowNetSimple) and including a layer that correlates feature vectors at different image locations.
- optical estimation requires per-pixel localization and finding correspondence between two input images. Not only learning image feature representations, but also match them at different locations in two images.
- original methods of sliding window CNN have the problems of: high computational costs, disallowing to account for global output properties. Another approach is to upsample all feature maps to the desired full resolution and stack them together, resulting in a concatenated per-pixel feature vector.
### 2.3 Our improvement

- create a new dataset: Flying Chair.
- Some original refining coarse depth map by [Eigen] training an additional network which gets the coarse prediction and the input image as input. [Long and Dosovitskiy] iteratively refine the coarse feature maps with upconvolutional layers. We integrate them together, we upconvolve not only the final coarse feature maps, but also some high level information. and we concatenate the upconvolutional result with the features from the contractive part of the network.

### 2.4 Method

#### 2.4.1 Simple

#### 2.4.2 Correlation

- separate processing stream for them and combine them at a later stage. The network is constrained to first produce meaningful representation of 2 images and then combine them at a higher level. Just like firstly extract features from patches of both images and compares those feature vectors.
- calculate method: consider only a single comparison of two patches. The ’correlation’ of two patches centered at $\mathbf{x1}$ in the first map and $\mathbf{x2}$ in the second map is then defined as
- ![image-20191107005323213](C:\Users\lxa\AppData\Roaming\Typora\typora-user-images\image-20191107005323213.png)
- To reduce the number of calculations, we limit the maximum displacement and introduce striding in both feature maps. Given a maximum displacement d, for each location x1 we compute correlations c(x1; x2) only in a neighborhood of size D := 2d + 1, by limiting the range of x2. We use strides s1 and s2, to quantize x1 globally and to quantize x2 within the neighborhood centered around x1.
- **[Question : This means we obtain an output of size $(w * h * D^2)$]**

#### 2.5.3 Refinement

![image-20191107011528047](C:\Users\lxa\AppData\Roaming\Typora\typora-user-images\image-20191107011528047.png)

- Original CNN: extract high-level abstract features of images, but not per-pixel predictions.
- The main ingredient are ‘upconvolutional’ layers, consisting of unpooling (extending the feature maps, as opposed to pooling) and a convolution. To perform the refinement, we apply the ‘upconvolution’ to feature maps, and concatenate it with corresponding feature maps from the ’contractive’ part of the network and an upsampled coarser flow prediction (if available). This way we preserve both the high-level information passed from coarser feature maps and fine local information provided in lower layer feature maps. 但相比bilinear upsampling这种computationally less expensive的方法没什么显著提升。另一种叫做variational refinement的方法，在动作较小时可以显著提升，在动作较大时可以使光流更加smooth，降低EPE（是否可以把这种方法应用在别的网络的upsample？）[Question: can't understand this method]


## 3 FlowNet2.0

### 3.1 Structure of FlowNet2.0

![image-20191107015602698](C:\Users\lxa\AppData\Roaming\Typora\typora-user-images\image-20191107015602698.png)

### 3.2 Our improvement

- 增加了一个具有3维运动的数据库FlyingThings3D。相比于FlyingChair中的图像只具有平面变换，FlyingThings3D中的图像具有真实的3D运动和亮度变化，按理说应该包含着更丰富的运动变换信息，利用它训练出的网络模型应该更具鲁棒性。然而实验发现，训练结果不仅与数据种类有关，还与数据训练的顺序有关。实验表明，先在FlyingChair上按S_long策略，再在FlyingThings3D上按S_fine策略后，所得结果最好。单独在FlyingThing3D上训练的结果反而下降。文中给出了解释是尽管FlyingThings3D比较真实，包含更丰富的运动信息，但是过早的学习复杂的运动信息反而不易收敛到更好的结果，而先从简单的运动信息学起，由易到难反得到了更好的结果。同时，结果发现FlowNetC的性能要高于FlowNetS。

- 利用堆叠的结构对预测结果进行多级提升。实验结果证明在FlowNetC的基础上堆叠FlowNetS，当以每个FlowNet为单位逐个进行训练时，得到的结果最优。也就是说在训练当前FlowNet模块时，前面的FlowNet模块参数均为固定状态。此外，发现后续的堆叠FlowNet模块，除了输入I_1、I_2外，再输入前一模块的预测光流W_i，图像I_2经预测W_i的变换图像I_2(w_i)以及误差图像|I_1-I_2(W_i)|后，可以使新堆叠的FlowNet模块专注去学习I_1与I_2(W_i)之间剩下的运动变换，从而有效的防止堆叠后的网络过拟合。实验表明，当以FlowNetC为基础网络，额外堆叠两个FlowNetS模块后，所得结果最好。

- 针对小位移的情况引入特定的子网络进行处理。针对小位移情况改进了FlowNet模块的结构，首先将编码模块部分中大小为7x7和5x5的卷积核均换为多层3x3卷积核以增加对小位移的分辨率。其次，在解码模块的反卷积层之间，均增加一层卷积层以便对小位移输出更加平滑的光流预测。FlowNet2-SD。在训练数据的选择上，针对小位移，又重新合成了以小位移为主的新的数据库ChairsSDHom,并将此前的堆叠网络FlowNet2-CSS在ChairsSDHom和FlyingThings3D的混合数据上继续微调训练，将结果网络表示为FlowNet2-CSS-ft-sd。

  最后，再利用一个新的小网络对FlowNet2-CSS-ft-sd的结果和FlowNet2-SD的结果进行融合，并将整个网络体系命名为FlowNet2。

- 小位移情况下，FlowNet2-CSS的光流预测噪声非常大，而FlowNet2-SD的输出非常光滑，最后融合结果充分利用了FlowNet2-SD的结果。大位移情况下，FlowNet2-CSS预测出了大部分运动，而FlowNet2-SD则丢失了大部分运动信息，最后融合结果又很好的利用了FlowNet2-CSS的结果。

​       综上，FlowNet2-CSS与FlowNet2-SD做到了很好地互补，共同保证了FlowNet2在各种情况下的预测质量。

【我们能否利用这种思想呢？】


