# Learning Human Optical Flow

## Michael J. Black

## 1 Introduction

### 1.1 Original Problems of Optical Flow

- Generic, low-level.
- Human flow: humans are complex, articulated, vary in shape, size and appearance. Human move quickly, self occlude and adopt a wide range of poses.

### 1.2 What has our methods done && How we do this

- A flow algorithm especially for humans and their motions.
- Create a large and realistic dataset of humans moving in virtual worlds with ground truth optical flow.(images and flow fields)
- A CNN based on SPyNet, extend it into a end-to-end trainable one. 【Question: Why is the SPyNet a good choice? **Compact and Computationally efficient**. Can we shift to another one? (Have to study the structure of SPyNet)】
- Utilize SMPL Model. 【Question: What's the meaning of 'using motion capture data', how we represent the statistics of natural human motion】【Question: What's the details of spatial pyramid】【Question: Why do we use SMPL model? Will Blender model works better? Consider the principle of optical flow, the key-point representation is closely related to optical flow, but it's vulnerable to light shifting. Maybe a surface representation could utilize some other information to tackle this?】
- A small CNN and is very fast.

## 3 The Human Flow Dataset

![image-20191103104653134](C:\Users\lxa\AppData\Roaming\Typora\typora-user-images\image-20191103104653134.png)

### 3.1 Body model and Rendering Engine

- SMPL Model, use pose and shape parameters to model realistic articulated motions.
- Vector pass, produce motion blur and motion of every pixels.

## 4 Learning

![image-20191103110504273](C:\Users\lxa\AppData\Roaming\Typora\typora-user-images\image-20191103110504273.png)


### 4.1 Architecture

- SPyNet: different convnets at different levels of an image pyramid, have to train independently. 【Question: What's fixed bilinear operators, why the downsampling and upsampling need a CNN?】【Question: What's the purpose of warping?】