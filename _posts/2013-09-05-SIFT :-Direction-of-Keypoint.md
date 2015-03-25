---
layout: post
category : image processing
tagline: "SIFT: KeyPoint方向定位"
tags : [SIFT, direction]
use_latex : true
use_toc : true
---

## KeyPoint方向定位
1. [尺度空间](http://jiehust.github.io/image processing/2013/09/01/SIFT :-Scale-Space/)
2. [特征点定位](http://jiehust.github.io/image processing/2013/09/03/SIFT :-Find-Keypoints/)
3. [特征点方向](http://jiehust.github.io/image processing/2013/09/05/SIFT :-Direction-of-Keypoint/)
3. [描述子生成](http://jiehust.github.io/image processing/2013/09/07/SIFT :-Descriptor/)

### 计算邻域梯度方向和幅值

为了实现图像旋转的不变性，需要根据检测到的特征点的局部图像结构求得一个方向基准。我们使用图像梯度的方法求取该局部结构的稳定方向。对于己经检测到特征点，我们知道该特征点的尺度值σ，因此根据这一尺度值，在 GSS 中得到最接近这一尺度值的高斯图像。然后使用有限差分，计算以特征点为中心，以 3X1.5σ 为半径的区域内图像梯度的幅角和幅值，如下图所示。幅角和幅值计算公式加下:

<img class="aligncenter" src="{{BASE_PATH}}/assets/img/sift_feature_direction.jpg" alt="Drawing" style="width: 420px;" align="center"/>

### 计算梯度方向直方图

在完成特征点邻域的高斯图像的梯度计算后，使用直方图统计邻域内像素的梯度方向和幅值。梯度方向直方图的横轴是梯度方向角，纵轴是梯度方向角对应的（带高斯权重）梯度幅值累加值。梯度方向直方图将。0°~360°的范围，分为36个柱，每10°为一个柱。直方图的峰值代表了该特征点处邻域内图像梯度的主方向，也即该特征点的主方向，如下图所示。

<img class="aligncenter" src="{{BASE_PATH}}/assets/img/sift_feature_direction1.png" alt="Drawing" style="width: 450px;" align="center"/>

绿色格点代表邻域范围，蓝色圆圈代表格点的高斯权重（稍后介绍），黑色箭头指向代表梯度方向，箭头长度代表梯度幅值。右边为梯度方向直方图（36柱，每柱代表10°，上图只显示了8柱）。获得梯度方向直方图的步骤如下：

- 生成领域各像元的高斯权重。其中高斯函数方差为该特征点的特征尺度σ的1.5倍。形式如下，其中（i,j）为该点距离特征点的相对位置，如上图，左上角点像元距离特征点（0,0）（即中心点）的相对位置坐标为（-4,-4），同理，右下角像元为（4,4）。

$$ w\_{i,j}=e^{frac{-(i^{2}+j^{2})}{2*(1.5\sigma)^{2}}} $$

- 遍历邻域（绿色）中每个点，判断其梯度方向，将其加入相应的梯度方向直方图中，加入量为其梯度幅值 * $w\_{i,j}$ ，例如左上角(-4,-4)的点，其梯度为方向为 25°，梯度幅值为 mag，我们将其加入到 hist[2] 中（假设 hist[0] 为 0°~10° 的直方柱，hist[1] 为 10°~20° 的直方柱，以此类推至 hist[35]为350°~360°）。加入的量为 mag* $w\_{-4,-4}$，即 hist[2] = hist[2] + mag* $w\_{-4,-4}$。直至遍历整个邻域，统计出该特征点出的梯度方向直方图。

- 平滑直方图。对上一步得出的直方图进行平滑，得到最终的梯度方向直方图。OpenCV 中使用的 (1/16) * [1,4,6,4,1] 的高斯卷积和对直方图进行平滑处理，而 vlfeat 中使用了6次，邻域大小为3的平均处理，即 hist[i] = (hist[i-1]+hist[i]+hist[i+1])/3。

**问题1： 为什么每个点梯度幅值要使用高斯权重？**

答：由于SIFT算法只考虑了尺度和旋转的不变性，并没有考虑仿射不变性。通过对各点梯度幅值进行高斯加权，使特征点附近的梯度幅值有较大的权重，这样可以部分弥补因没有仿射不变性而产生的特征点不稳定的问题。

### 确定特征点方向

有了梯度方向直方图之后，找到直方图中最大的值，则认为该方向为该特征点的主方向，如存在另一个方向大于最大值的80％，则认为该方向为该特征点的辅方向。一个特征点可能会有多个方向（一个主方向，一个以上的辅方向），这可以增强匹配的鲁棒性。具体而言，就是将该特征点复制成多份特征点（除了方向θ不同外，x,y,σ都相同）。

【Note】在OpenCV中，若辅方向除了满足大于最大值80％外，还必须是局部最大值，即 hist[i] > hist[i-1] && hist[i] > hist[i+1]。

<img class="aligncenter" src="{{BASE_PATH}}/assets/img/sift_feature_direction2.jpg" alt="Drawing" style="width: 150px;" align="center"/>

通常离散的梯度方向直方图，可以通过插值拟合处理，这样可以得到更精确的方向角度值。

<img class="aligncenter" src="{{BASE_PATH}}/assets/img/sift_feature_direction3.jpg" alt="Drawing" style="width: 300px;" align="center"/>

经过上述过程，我们特征点的所有量（x,y,σ,θ）都已经已经求得，其中位置（x,y）、尺度 σ 都是在上一节中求得，而特征点方向 θ 是通过特征点邻域直方图求得。

--- 
参考资料：

1. [David G. Lowe Distinctive Image Features from Scale-Invariant Keypoints](http://www.cs.ubc.ca/~lowe/papers/ijcv04.pdf) 
2. 王永明 王贵锦 《图像局部不变性特征与描述》

