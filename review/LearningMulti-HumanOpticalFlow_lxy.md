# **Learning Multi-Human Optical Flow**

## Michael J. Black

### 1. Main idea

They use their own dataset to obtain more accurate 2D motion estimates for human bodies. To improve the accuracy, they create a more detailed dataset by adding details of human hand motions.

They call their dataset MHOF. And their results are reached by comparisons of different combinations of SHOF, MHOF and SPyNet, PWC-Net.

This innovation is the first bringing up of Muti-Human Optical Flow, as what we talked in the past were all achievements of Single-Human Optical Flow.

### 2. Thoughts

Since they haven't propose a new net framework, we can try coming up with a new net framework and run their dataset. If we obtain better results using our net, it'll be our innovation.

If we can improve our dataset and make it more realistic or adding more details, we'll possibly obtain better results, but it lacks creativity.

They use the same indoors background like others. Is it possible if we create our new outdoors background or indoors background with some tiny changes in light but still obtain good results by using certain ways to reduce the impact?