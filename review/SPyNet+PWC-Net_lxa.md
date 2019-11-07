# PWC-Net && SPyNet

## Deqing SUN; Michael J. Black

## 1 Introduction

### 1.1 Structure

 ![img](file:///C:\Users\lxa\Documents\Tencent Files\1021289005\Image\C2C\Image1\E]FZH%N`N]4@VSP358~C@6H.png) 

## 2 SPyNet

### 2.1 Structure of SPyNet

![image-20191104230636987](C:\Users\lxa\AppData\Roaming\Typora\typora-user-images\image-20191104230636987.png)

### 2.2 Introduction

- Spatial-Pyramid formulation method: estimate motions in a coarse-to-fine approach by warping one image of a pair at each pyramid level by the current flow estimate and computing an update to the flow. 
- One assumption at the top level of pyramid: motions between frames are smaller than a few pixels and the convolutional filters can learn meaningful temporal structure. (because we've downsampled the image, can prevent the big motion by downsample the image?)
- Instead of minimizing function, we learn a CNN to predict the flow increment at that level and add this to the flow output of the network above.
- **Still existing problem: large motions of small or thin objects are difficult to capture with a pyramid representation.**

### 2.3 Original Problems

- classical formulation: ill assumptions, brightness constancy and spatial smoothness. Limit performance. can't solve motion too large, filter can't perceive it.
- Ideally such a network would learn to solve the correspondence problem (short and long range), learn filters relevant to the problem, learn what is constant in the sequence, and learn about the spatial structure of the flow and how it relates to the image structure. The first attempts are promising but are not yet as accurate as the classical methods.

### 2.4 Our improvement

- better performance, real-time, quicker, less memory cost.
- Computing optical flow contains 2 problems: long-range correlations,detailed sub-pixel optical flow and precise motion boundaries. FlowNet: learn both of them at once. We use deep learning to solve the latter one and existing method to solve the former one. (Referring to FlowNet and FlowNet2.0)

### 2.5 Method

#### 2.5.1 Spatial Sampling

- downsampling and upsampling, warping operation. $w(I, V)$ means warping the image $I$ with the flow field $V$

#### 2.5.2 Inference

- train K + 1 networks,
- $v_{k}=G_{k}\left(I_{k}^{1}, w\left(I_{k}^{2}, u\left(V_{k-1}\right)\right), u\left(V_{k-1}\right)\right)$
- $V_{k}=u\left(V_{k-1}\right)+v_{k}$

#### 2.5.3 Training

- gt: $\hat{v}_{k}=\hat{V}_{k}-u\left(V_{k-1}\right)$
- minimize EPE: $\frac{1}{m_{k} n_{k}} \sum_{x, y} \sqrt{\left(v_{k}^{x}-\hat{v}_{k}^{x}\right)^{2}+\left(v_{k}^{y}-\hat{v}_{k}^{y}\right)^{2}}$
- where the $m_{k} \times n_{k}$ is the image dimension at level $k$ and the $x$ and $y$ superscripts indicate the horizontal and vertical components of the flow vector.
- each layer only need to update a little bit, so it's simple. each of them has 5 conv layers.
- Each convolutional layer is followed by a Rectified Linear Unit (ReLU), except the last one. We use $7 \mathrm{x} 7$ convolutional kernels for each layer; these worked better than smaller filters. The number of feature maps in each convnet,$G_{k}$ are $\{32,64,32,16,2\} .$ The image $I_{k}^{1}$ and the warped image $w\left(I_{k}^{2}, u\left(V_{k-1}\right)\right)$ have 3 channels each (RGB). The upsampled flow $u\left(V_{k-1}\right)$ is 2 channel (horizontal and vertical. We stack image frames together with the upsampled flow to form an 8 channel input to each $G_{k} .$ The output is 2 channel flow corresponding to velocity in $x$ and $y .$


## 3 PWC-Net

### 3.1 Structure of PWC-Net

![image-20191105092330698](C:\Users\lxa\AppData\Roaming\Typora\typora-user-images\image-20191105092330698.png)

### 3.2 Introduction

- most top-performing method utilizes energy minimization, requires much time.

- **FlowNet is based on a generic U-Net CNN architecture, but with a poor performance.  **

- FlowNet2.0 stack several FlowNetS and FlowNetC together and the performance is equal to state of art, but has the problem of overfitting and big memory cost.

- SPyNet uses spatial pyramid network to estimate motion from 1st image to the warped image, which is quite small. Hence, we only need a small network. It's a tradeoff between size and accuracy.

- **SPyNet combines classical principles with CNNs, but partial use of classical principles. Traditional methods pre-process the raw images to extract features that are invariant to shadows or lighting changes.**  

- cost volume is a more discriminative representation of the disparity (1D flow) than raw images or features. While constructing a full cost volume is computationally prohibitive for real-time optical flow estimation, this work constructs a “partial” cost volume by limiting the search range at each pyramid level.


### 3.3 Method

- As raw images are variant to shadows and lighting shifts, we replaces the fixed images pyramid with learnable feature pyramids.
- cost volume is a more discriminative representation of the optical flow than raw images. we use cost volume layer to generate the cost volume. cost volume layer and warping layers have no trainable parameters to reduce the model size.
- use context network to exploit contextual information to refine the optical flow.

#### 3.3.1 Feature pyramid extractor

- Feature pyramid extractor. Given two input images $\mathbf{I}_{1}$ and $\mathbf{I}_{2},$ we generate $L$ -level pyramids of feature representations, with the bottom (zeroth) level being the input images, i.e., $\mathbf{c}_{t}^{0}=\mathbf{I}_{t}$. To generate feature representation at the $l$ th layer, $\mathbf{c}_{t}^{l},$ we use layers of convolutional filters to downsample the features at the $l-1$ th pyramid level, $\mathbf{c}_{t}^{l-1},$ by a factor of $2 .$ From the first to the sixth levels, the number of feature channels are respectively $16,32,64,96,128,$ and $196 .$

#### 3.3.2 Warping layer

- Warping layer. At the $l$ th level, we warp features of the second image toward the first image using the $\times 2$ upsampled flow from the $l+1$ th level:
$$
\mathbf{c}_{w}^{l}(\mathbf{x})=\mathbf{c}_{2}^{l}\left(\mathbf{x}+\operatorname{up}_{2}\left(\mathbf{w}^{l+1}\right)(\mathbf{x})\right)
$$

- where $\mathrm{x}$ is the pixel index and the upsampled flow $\mathrm{up}_{2}\left(\mathrm{w}^{l+1}\right)$ is set to be zero at the top level. We use bilinear interpolation to implement the warping operation. For non-translational motion, warping can compensate for some geometric distortions and put image patches at the right scale.

#### 3.3.3 Cost volume layer

- Next, we use the features to construct a cost volume that stores the matching costs for associating  a pixel with its corresponding pixels at the next frame. We define the matching cost as the correlation between features of the first image and warped features of the second image:
$\mathbf{c v}^{l}\left(\mathbf{x}_{1}, \mathbf{x}_{2}\right)=\frac{1}{N}\left(\mathbf{c}_{1}^{l}\left(\mathbf{x}_{1}\right)\right)^{\top} \mathbf{c}_{w}^{l}\left(\mathbf{x}_{2}\right)$
where $T$ is the transpose operator and $N$ is the length of the column vector $\mathbf{c}_{1}^{l}\left(\mathbf{x}_{1}\right) .$ For an $L$ -level pyramid setting, we only need to compute a partial cost volume with a limited range of $d$ pixels, $i . e .,\left|\mathrm{x}_{1}-\mathrm{x}_{2}\right|_{\infty} \leq d .$ A one-pixel motion at
the top level corresponds to $2^{L-1}$ pixels at the full resolution images. Thus we can set $d$ to be small. The dimension of the $3 \mathrm{D}$ cost volume is $d^{2} \times H^{l} \times W^{l},$ where $H^{l}$ and $W^{l}$ denote the height and width of the $l$ th pyramid level, respectively.

#### 3.3.4 optical flow estimator

- It is a multi-layer CNN. Its input are the cost volume, features of the first image, and upsampled optical flow and its output is the flow $w^l$ at the $l$th level. The numbers of feature channels at each convolutional layers are respectively 128, 128, 96, 64, and 32, which are kept fixed at all pyramid levels. The estimators at different levels have their own parameters instead of sharing the same parameters. This estimation process is repeated until the
desired level, $l_0$.

#### 3.3.5 Context network

- enlarge the receptive field size of each output unit at the desired pyramid level. It takes the estimated flow and features of the second last layer from the optical flow estimator and outputs a refined flow.
- use dilated convolutions, from bottom to top, the dilation constants are 1, 2, 4, 8, 16, 1, and 1.

#### 3.3.6 Training loss

- learnable parameters: feature pyramid extractor and optical flow estimator
- $\mathcal{L}(\Theta)=\sum_{l=l_{0}}^{L} \alpha_{l} \sum_{\mathbf{x}}\left|\mathbf{w}_{\Theta}^{l}(\mathbf{x})-\mathbf{w}_{\mathrm{GT}}^{l}(\mathbf{x})\right|_{2}+\gamma|\Theta|_{2}$
- 【Question: fine-tuning?】
- ![image-20191105140900372](C:\Users\lxa\AppData\Roaming\Typora\typora-user-images\image-20191105140900372.png)