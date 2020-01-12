## Title

EasyMesh: An Efﬁcient Method to Reconstruct 3D Mesh From a Single Image

## Summary

在单视图重建任务中，直接将输入的 RGB 图像转换为 Shape 是及其困难的。

本文利用 RGB 图像构建 Geometry Image (本质是PointCloud）再进行 3D Reconstruction

主要由两个部分构成：

1、提取轮廓 

- 使用现成的语义分割网络 Deeplab V3+ 

2、利用提取的轮廓通过编解码构造 Geometry Image，并通过 Discriminator 进行判别，最终构造为 Mesh

- 三重残差块作 Encoder 提取轮廓特征
- 利用双线性插值和反卷积解码构建 Geometry Image

## Theory

GeometryImage 实际上是 PointCloud 的二维形式。对 GeometryImage 进行操作可以避免三维卷积昂贵的代价

整篇文章本质上还是 Point-based，只不过利用了 GeometryImage 减少了计算代价

## Implement

####  Generator 

![image-20191218115953363](/Users/bear/Library/Application Support/typora-user-images/image-20191218115953363.png)

生成器使用修改后的 Chamfer Distance 作为 Loss，与原本的 CD 相比减少了计算量
$$
\begin{aligned} L_{C D}\left(G_{p r e}, G_{g t}\right) &=\sum_{x \in G_{p r e}} \min _{y \in G_{g t} \cap N(x)}\|x-y\|_{2}^{2} \\ &+\sum_{y \in G_{g t}} x \in G_{p r e} \cap N(y) \end{aligned}
$$

$$
\begin{array}{l}{\text { where } G_{p r e ~ a n d ~} G_{g t} \text { are the predicted and ground-truth geometry images, respectively. The neighborhood } N(x) \text { of }} \\ {\text { point } x \text { in geometry image is defined as } K \times K \text { grid centered at } x \text {  }}\end{array}
$$

同时为了使得 PointCloud 生成的 Surface smooth，再用 Normal 作为损失保证 Normal 的质量
$$
L_{n o r m a l}=\sum_{i}<\overrightarrow{n_{i}}, \sum_{j \in N(i)}\left(\vec{p}_{i}-\overrightarrow{p_{j}}\right)>
$$

$$
\text { where } \vec{n}_{i} \text { is the ground-truth normal at point } p_{i}
$$

最后再对 EdgeLength 正则化避免离群点
$$
L_{e d g e}=\sum_{x} \sum_{y \in N(x)}\|x-y\|_{2}^{2}
$$
最终生成器的最终 Loss 为以上三个相加
$$
L_{m e s h}=L_{C D}+L_{n o r m a l}+L_{e d g e}
$$

##### Symmetry Padding

利用上述 Loss 生成的 image 在四角效果不好，文章根据折叠理论 (修改后的 Geometry Image 提出的方法)，提出反对称填充

对最终的 image 转置元素构造新的边

![image-20191218142401523](/Users/bear/Library/Application Support/typora-user-images/image-20191218142401523.png)

#### Discriminator

![image-20191218141618391](/Users/bear/Library/Application Support/typora-user-images/image-20191218141618391.png)

Discriminator 使用 Loss 如下
$$
L_{G A N}=E_{y \sim p_{y}} \log (D(y))+E_{s \sim p_{s}} \log (1-D(G(s)))
$$

$$
\begin{array}{l}{\text { where } y \text { denotes the ground-truth geometry image, } s \text { means the input silhouette image, } G(s) \text { denotes the synthesized }} \\ {\text { geometry image, and } D(\bullet) \text { represents the probability value that the discriminator outputs. }}\end{array}
$$

##### Viewpoint estimation

- 相机共有三个参数 camera location，direction of camera, up vector 
	- 后两个参数直接固定。。。  所以这个参数估计感觉没啥意义
	
	- 前一个参数通过 VGG 来获取，Loss 如下: 
		$$
		L_{v p}=E_{v \sim V_{p r e}, u \sim V_{g t}}\|v-u\|^{2}
		$$
		
		$$
		\text { where } v \text { is the predicted viewpoint and } u \text { is the corresponding ground truth. }
		$$

##### Silhouette image re-rendering

为了提高 3D object 的质量，将最终的 Geometry Image 结果转化为PointCloud 再对 Image 进行增强

首先映射 PointCloud 上的点到 image 坐标系上
$$
\begin{array}{l}{\text {  Given the }} {\text { rotation matrix } R_{i} \text { and transformation matrix } t_{i} \text { of viewpoint } i, \text { each point in the canonical coordinate system can be }} \\ {\text { mapped to the image coordinate by }}\end{array}
$$

$$
x=K\left(R_{i} p+t_{i}\right)
$$

$$
\begin{array}{l}{\text { where } K \text { is the camera intrinsic matrix, } p \text { is the } 3 \text {D canonical coordinate of a point, } x \text { denotes }\left(x_{c}, y_{c}, z_{c}\right) \text { in which }} \\ {\left(x_{c}, y_{c}\right) \text { is the corresponding coordinate in } 2 D \text { image plane, and } z_{c} \text { depth value }} \end{array}
$$
之后获取轮廓
$$
S_{(x, y)}=\left\{\begin{array}{ll}{\min \left(1, z_{c}\right),} & {\text { if } z_{c}>0} \\ {0,} & {\text { if } z_{c}=0}\end{array}\right.
$$
利用轮廓构造 Loss
$$
L_{s i l}=E_{x, y, i} M(x, y)\left\|S_{p r e, i}(x, y)-S_{g t, i}(x, y)\right\|
$$

$$
\begin{array}{l}{\text { where } S_{p r e, i} \text { and } S_{g t, i} \text { are the silhouettes rendered from the predicted and ground-truth point clouds, respectively, at }} \\ {\text { viewpoint } i, \text { and the total number of viewpoints is } N \text { . We punish the "empty" pixels that should have been inside the }} \\ {\text { outline of object by adding a mask } M, \text { in which the weight value is selected as } 3 \text { if } S(x, y)=1, \text { and } 1 \text { if } S(x, y)=0 \text { . }}\end{array}
$$

##### Surface reconstruction from geometry image

- 直接通过 Geometry Image 上点间的连接关系进行 Reconstruction

##### Final Loss

结合以上所有子过程，最终的 Loss 如下：
$$
L=\lambda_{1} L_{m e s h}+\lambda_{2} L_{G A N}+\lambda_{3} L_{v p}+\lambda_{4} L_{s i l}
$$
四个 $\lambda$ 是自定义权重，文中设定 $
\lambda_{1}=1, \lambda_{2}=0.1, \lambda_{3}=1, \lambda_{4}=30
$

## Experiment and Evaluation

使用指标 average point-wise Euclidean distance 

第一个部分用 PASCAL VOC 2012 进行训练，第二部分用 ShapeNet 训练

1. Reconstruction on multiple categories

   直接利用 ShapeNet 各个类别的 RGB 进行 Reconstruction

   ![image-20191218145537459](/Users/bear/Library/Application Support/typora-user-images/image-20191218145537459.png)

2. Reconstruction on real-world images

   利用 PASCAL 3D+ 及互联网上的照片进行 Reconstruction

   ![image-20191218145550230](/Users/bear/Library/Application Support/typora-user-images/image-20191218145550230.png)

3. Verifying the effectiveness of the discriminator

   利用 PointNet 及 ResNet 替代 Discriminator 验证文章中 Discriminator 的有效性

   ![image-20191218145617174](/Users/bear/Library/Application Support/typora-user-images/image-20191218145617174.png)

4. Ablation study

   取消某一个子过程对比取消后的结果从而确定每个子过程的作用

   ![image-20191218145629807](/Users/bear/Library/Application Support/typora-user-images/image-20191218145629807.png)

## Notes

如前面所说，这篇文章本质上是 Point-based ，比较有意思的地方还是转化为 GeometryImage 再进行相关操作 (我不知道这个方法是不是第一次被人这样用。。。)

除此之外整个结构用了 GAN 的思想并根据生成结果的问题 "哪里不行补哪里"，所以有了许多的子过程及对应的 Loss
