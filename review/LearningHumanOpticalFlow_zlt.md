### 主要任务
提出了专门针对人体运动的算法并标明比一般光流算法表现更好。做了一个新的图像序列的ground truth数据集，用CNN估计flow fields，输入是image pairs. 优化了SpyNet使fully end-to-end 可训练。  

原有datasets和benchmarks对人体运动任务是不充分的，一个主要原因在于人体运动在实际场景中很难准确捕获，因此做了一个新的针对人体运动的数据集。  

使用SMPL body model来生成人体形状，并将他们随机置于室内外背景，并simulate人体运动。基于spatial pyramids训练网络，并评价网络估计人体运动的性能。  

使用SpyNet来训练光流算法，因为它更compact，计算更有效。网络大小比较小仅7.8MB的参数，可运用于嵌入式应用。   

### The Human Flow Dataset
SMPL模型是一种参数化人体模型。这种方法可以模拟人的肌肉在肢体运动过程中的凸起和凹陷。因此可以避免人体在运动过程中的表面失真，可以精准的刻画人的肌肉拉伸以及收缩运动的形貌。该方法中β和θ是其中的输入参数，其中β代表是个人体高矮胖瘦、头身比等比例的10个参数，θ是代表人体整体运动位姿和24个关节相对角度的75个参数。  

使用SMPL的模型来生成了很多的人体外形，使用Blender作为渲染引擎生成了很多的图像帧和光流，也包含了UV appearance map来改变skin tone, face feature 和clothing texture等。SPyNet可以再看看专门介绍Spatial Pyramid Network的论文。

### Learning
网络的学习使用SPyNet的改进版，使用了warping operator，w，建立了fully differentiable spatial pyramid architecture， 并通过end-to-end训练来最小化End Point Error(EPE).论文的实验结果表明该方法比原先的state-of-the-art算法要更优越，更快，且sharp details如人体的手脚都能得到更好的结果。

### 反思
论文提出计划对更微小的运动如脸和手进行建模，并对contining multiple, interacting people 和更复杂的3D场景进行建模，并增加3D clothing和accessories.  

论文中我对SPyNet和其改进的理解还不够深刻，并且一些实践的细节不是很清楚，对Spatial Pyramid Network的论文还需要再看一下。