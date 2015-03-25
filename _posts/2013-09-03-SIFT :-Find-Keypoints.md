---
layout: post
category : image processing
tagline: "SIFT: KeyPoint找寻、定位与优化"
tags : [SIFT, keypoint, hessian]
use_latex : true
use_toc : true
---

## KeyPoint找寻、定位与优化
1. [尺度空间](http://jiehust.github.io/image processing/2013/09/01/SIFT :-Scale-Space/)
2. [特征点定位](http://jiehust.github.io/image processing/2013/09/03/SIFT :-Find-Keypoints/)
3. [特征点方向](http://jiehust.github.io/image processing/2013/09/05/SIFT :-Direction-of-Keypoint/)
3. [描述子生成](http://jiehust.github.io/image processing/2013/09/07/SIFT :-Descriptor/)

### KeyPoint找寻

极值的检测是在 DoG 空间进行的，检测是以当前点为中心，3pixel X 3pixel X 3pixel 的立方体为邻域，判断当前点是否为局部最大或最小。如下图所示，橘黄色为当前检测点，绿色点为其邻域。因为要比较当前点的上下层图像，所以极值检测从 DoG 每层的第 2 幅图像开始，终止于每层的倒数第 2 幅图像（第 1 幅没有下层，最后 1 幅没有上层，无法比较）。

<img class="aligncenter" src="{{BASE_PATH}}/assets/img/sift_keypoint.png" alt="Drawing" style="width: 300px;" align="center"/>

### KeyPoint定位

以上极值点的搜索时在离散空间中进行的，检测到的极值点并不是真正意义上的极值点。如下图所示，连续空间中极值与离散空间的区别。通常通过插值的方式，利用离散的值来插值，求取接近真正的极值的点。

<img class="aligncenter" src="{{BASE_PATH}}/assets/img/sift_keypoint1.jpg" alt="Drawing" style="width: 300px;" align="center"/>

对于一维函数，利用泰勒级数，将其展开为二次函数：

$$f(x)\approx f(0)+f'(0)x+1/2f''(0)x^{2}$$

对于二维函数，泰勒展开为：

$$ f(x,y)\approx f(0,0)+(\frac{\partial f}{\partial x}x+\frac{\partial f}{\partial y}y)+\frac{1}{2}(\frac{\partial^2 f}{\partial x^2}x^2 + 2\frac{\partial^2 f}{\partial x \partial y}xy + \frac{\partial^2 f}{\partial y^2}y^2) $$

矩阵表示为：

$$
f(\begin{bmatrix}x \\\\ y \end{bmatrix}) \approx f(\begin{bmatrix}0 \\\\ 0 \end{bmatrix})+\begin{bmatrix}\frac{\partial f}{\partial x} & \frac{\partial f}{\partial y} \end{bmatrix} \begin{bmatrix}x\\\\ y\end{bmatrix} +\frac{1}{2}\begin{bmatrix}x & y\end{bmatrix}  \begin{bmatrix}\frac{\partial^{2} f}{\partial x^{2}} & \frac{\partial^{2} f}{\partial x \partial y} \\\\  \frac{\partial^{2} f}{\partial x \partial y} & \frac{\partial^{2} f}{\partial y^{2}}  \end{bmatrix} \begin{bmatrix}x\\\\ y\end{bmatrix}
$$

矢量表示为：
$$f(\textbf{x})\approx f(\textbf{0})+\frac{\partial f}{\partial \textbf{x}})^T \textbf{x} + \frac{1}{2} \textbf{x}^T \frac{\partial^{2} f}{\partial \textbf{x}^{2}}\textbf{x} $$

求取 $f(\textbf{x})$ 的极值，只需求取 $\frac{\partial f}{\partial \textbf{x}}=0$。对于极值，$x,y,\sigma$ 三个变量，即为三维空间。利用三维子像元插值，设其函数为 $D(x,y,\sigma)$，令 $\textbf{x} = (x, y, \sigma)^{T}$，那么在第一节中找到的极值点进行泰勒展开为（式-1）如下：

$$
D(\textbf{x})=D+(\frac{\partial D}{\partial \textbf{x}})^T \textbf{x} + \frac{1}{2} \textbf{x}^T \frac{\partial^{2} D}{\partial \textbf{x}^{2}}\textbf{x} 
$$

其中 $D$ 为极值点的值，$(\partial D/\partial \textbf{x})^{T}$ 为在极值点各自变量的导数，$(\partial^{2} D/\partial \textbf{x}^{2})$ 为其在展开点相应的矩阵。对上式求导，另 $\partial D(\textbf{x})/\partial \textbf{x} = \textbf{0}$，结果如下式，对应的 $\hat{\textbf{x}}$ 向量即为真正极值点偏离插值点的量。求解得（式-2）如下：

$$
\hat{\textbf{x}}=-\frac{\partial^{2} D^{-1}}{\partial \textbf{x}^2} \frac{\partial D}{\partial \textbf{x}}
$$

最终极值点的位置即为插值点 $\textbf{x}+\hat{\textbf{x}}$，且多次迭代可以提高精度（一般为5次迭代）。

**问题 1：** 《图像局部不变性特征与描述》中提到，对于偏移量ˆx任何方向上偏移大于0.5的特征点，要删除该点。

对于Lowe的原文为：
> If the offset ˆx is larger than 0.5 in any dimension, then it means that the extremum lies closer to a different sample point. In this case, the sample point is changed and the interpolation performed instead about that point.

个人认为原作者并未说要删除此类点，只是说这个点偏移了，所以需要插值来进行替换。在个人研究的源码中，OpenCV sift的源码中，并未删除上述类型的点，在vlfeat的开源代码中，也未删除上述点。

### KeyPoint优化

对KeyPoint定位后，要剔除一些不好的KeyPoint，那什么是不好的KeyPoint的呢？

1. DoG响应较低的点，即极值较小的点。
2. 响应较强的点也不是稳定的特征点。DoG对图像中的边缘有较强的响应值，所以落在图像边缘的点也不是稳定的特征点。一方面图像边缘上的点是很难定位的，具有定位的歧义性；另一方面这样的点很容易受到噪声的干扰变得不稳定。

**对于第一种**，只需计算矫正后的点的响应值D(ˆx)，响应值小于一定阈值，即认为该点效应较小，将其剔除。将（式-2）带入（式-1），求解得：
$$
D(\hat{\textbf{x}})=D+ \frac{1}{2} \frac{\partial D^{T}}{\partial \textbf{x}} \hat{\textbf{x}}
$$
在Lowe文章中，将 $\left | D(\hat{\textbf{x}}) \right | < 0.03$（图像灰度归一化为[0,1]）的特征点剔除。

**对于第二种**，利用 Hessian 矩阵来剔除。一个平坦的 DoG 响应峰值在横跨边缘的地方有较大的主曲率，而在垂直边缘的地方有较小的主曲率。主曲率可以通过 2×2 的 Hessian 矩阵 H 求出：

$$
H(x,y)=\begin{bmatrix}
D\_{xx}(x,y) & D\_{xy}(x,y)\\\\ 
D\_{yx}(x,y) & D\_{yy}(x,y)
\end{bmatrix}
$$

D 值可以通过求临近点差分得到。H 的特征值与 D 的主曲率成正比，具体可参见 Harris 角点检测算法。为了避免求具体的值，我们可以通过 H 将特征值的比例表示出来。令 $\alpha =\lambda\_{max}$ 为最大特征值，$\beta =\lambda\_{min}$ 为最小特征值，那么：

$$Tr(H)=D\_{xx}+D\_{yy}=\alpha +\beta $$
$$Det(H)=D\_{xx}D\_{yy}-D\_{xy}D\_{yx} = \alpha \cdot \beta $$

$Tr(H)$ 为矩阵 H 的迹，$Det(H)$ 表示 H 的行列式。令 $\gamma  = \alpha / \beta$ 表示最大特征值与最小特征值的比值，则有：

$$ \frac{Tr(H)^{2}}{Det(H)}=\frac{(\alpha+\beta)^{2}}{\alpha \cdot \beta}=\frac{(\gamma+1)^{2}}{\gamma}$$

上式与两个特征值的比例有关。随着主曲率比值的增加，$\frac{(\gamma+1)^{2}}{\gamma}$ 也会增加。我们只需要去掉比率大于一定值的特征点。Lowe 论文中去掉 $\gamma=10$ 的点。

--- 
参考资料：

1. [David G. Lowe Distinctive Image Features from Scale-Invariant Keypoints](http://www.cs.ubc.ca/~lowe/papers/ijcv04.pdf) 
2. 王永明 王贵锦 《图像局部不变性特征与描述》


