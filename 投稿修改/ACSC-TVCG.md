# Reviewer 3 (committee member)

## Section 3 should be clarified in several places. The description is rather long but often imprecise.


## The term "systematic error of SSL" appears several times in the paper. Could you explain what you mean under "systematic error"?
这个就是SSL的测量误差， 在Fig. 3和Section 3开头描述

## In algorithm 1: points such that diskKI<thresh are removed. I suspect that it is the contrary.
应该是>, 这里是typo

## Could you explain the two cases of equation 6?
公式6，构造棋盘格模型的解释

## What is the value of delta_{rho} which is guiding the random sampling of the point cloud? Since sampling is recognized as the most influential parameter in the ablation study, what is the influence of this value and how is it chosen?
重采样的密度是多少？

对于LiVox Horizon，是7，对于mid，是5

## The fact that eq 9 is the same as in [31] should be mentioned explicitly
与ILCC在loss上的区别是需要被详细说明
在ILCC中，是先将点云反射率二值化

而我们的方法是直接将反射率的大小作为某个点属于黑色/白色格子的隶属度

## As indicated by the authors at the end of section 4.5, the size of the target and environmental factors may influence the algorithm. The authors should explain what are the most influential factors.
棋盘格尺寸对结果的影响


## Determining how to match reflectance intensity and color features is an important problem. In this regard, Fig. 6 is supposed to show that the reflectance on the checkerboard and the black and white patterns show the same distribution. Since Fig. 6 only show the reflectance it is however hard to compare reflectance and color distribution. Note that mutual information could be a way to quantify the similarity of the two intensities.



## Eq (1) in section 3.1.3 is not clear to me. Z_{down} is a function of z, so eq (1) does not allow to define a unique Z_{down}. If we consider the minimum of this expression, this doesn't make much sense either since the first term (argmax) is in fact a constant of z. The way Z_{down} is computed has to be clarified.
Zdown是checkerboard下边缘与支撑杆的链接处的高度。一个对Eq.1直观的解释是，我们计算标定板点云在沿着高度方向的宽度分布构建了一个直方图，这个直方图的x轴是点所在位置的高度，y轴是这个位置的点云宽度。因为链接处是点云宽度变化最明显的位置，那么对于这个直方图来说，其梯度最大的位置的x值就是链接处的高度

## Still in 3.1.3, the authors propose to recover the checkerboard plane through an iterative refinement step. Starting from the estimated parameters, points are kept according to a decreasing point-to-plane distance (eq 3). By doing so, integrating points of the last point cloud is more selective (ie delta is smaller) than for the first point cloud. The authors should explain why some point clouds (the ones that were examined first ) are privileged over others.
最开始用全局的点云，拟合出的平面会和真实的checkerboard相近，但是偏差特别大的噪声会影响结果


# Reviewer 2 (reviewer)

## In 2.2, the authors revise targetless calibration methods. However, since they then stick to target-based calibration themselves, I believe this section could be significantly shortened. On the other hand, the claim that target-based calibration "is usually more precise and robust" should be questioned. If a target-less method is presented with a nice and high-contrast calibration pattern/object, would it not have a chance at performing equally well? I.e. how much of the precision and robustness is due to the high contrast and/or distinctiveness of the target shape that is typically offered, and how much of it is due to actual knowledge of the target?
有目标标定和无目标标定的区别

## In Section 3.1.1, the authors claim that "previous calibration methods [...] can only obtain rough feature extraction results" on the data under consideration. However, the authors never present or discuss features extracted by "previous calibration methods" in detail. After some integration time with the solid-state lidar, I'm not sure that just the different scanning pattern and subsequent point distribution will automatically lead to failures. For a conference or workshop paper, that's fine, but I think in a journal this has to be verified in experiments.
之前的工作为什么没有被展示
我们尝试过多个github上的实现

## If the authors consider the time-domain integration a major novel contribution of the paper, then a bit more discussion and explanation is needed for Algorithm 1. It appears that nearby points (disK_i *<* thresh) are removed, which intuitively sounds like the opposite of an outlier removal. Also, there are standard procedures for outlier removal, like the local outlier factor. Would they be applicable?
这里是笔误

## In the "Ideal plane fitting" in Section 3.1.3, a method is presented that appears to keep only points that are close enough to an ideal plane. With the iterative reduction of the threshold, it is not clear, how many points will be preserved after all, and the convergence properties are unclear. If this scheme is motivated by the literature, a reference is required, if it's something the authors came up with, a more detailed investigation would be appropriate.
平面拟合的问题，保留了多少点，有多少点需要被删除，这些都没有详细说明，如果这个来自其他的文章，需要加引用

## In eq. (9) the authors directly relate some known calibration pattern reflectance to measurements from the lidar sensor. Can these be directly compared, or would a mutual information be more appropriate? Also, if they can really be directly related, why did the authors chose the absolute difference instead of a more typical squared difference?
使用互信息或者直接比较是否会更合适

## The authors state that they use "L-BGFS [1]" for optimizing R and t. a) It's usually called L-BFGS, not L-BGFS, but I don't think the gentlemen would object to having their names reordered. b) For the 6 parameters the authors estimate, a standard BFGS would seem more than sufficient and I wonder, why they chose L-BFGS. c) If a squared difference was selected in eq. (9), wouldn't a standard Least Squares optimizer be even better than the selected first-order methods?
L-BFGS写错了；BFGS和L-BFGS一样的，为什么不用BFGS

我们用的是L-BFGS的实现，其和BFGS的效果类似，因此为了真实性，在论文中我们写的是使用了L-BGFS算法

## 1) and 2) are also based on well-known standard techniques. 3) is based on [31], which is also presenting a method for obtaining accurate 3D corner positions from the data taken with a Lidar, so novelty and originality over [31] are also not clear.

The accuracy of the presented method is compared with [31] and [34], which are calibration methods for ring-Lidar. The accuracy comparison presented in table 2 demonstrates better accuracy of the result with the presented method, but it is not clear why the presented method is more accurate than others. If there is a significant novel and original contribution comparing with those methods, the authors should clearly describe the contribution by pointing out the differences/advantages comparing with those methods. However, such differences/advantages are not clearly explained in this paper, but simply show the compared results in the experiment.



# Reviewer 1 (reviewer)

## minor aspects
- Fig 6 : lack of units
- Fig 10 : it is very hard to understand given that there are three axes.
- Notations hard to understand or not explained: Pb Pc, Pi, …, H
- Notation discrepancies eq. 7 / 9 : Sc^I, Pc^I turns into Is, Ip
- After eq (9) : “Gi is the closet corner to ˜pi” → “closest”
- Compared (...) in a controlled environment. The targetless-based method aims at finding natural patterns (mainly the line or orthogonal) ->”mainly lines and right angles”
- Section 4.5 : missing space: “the reprojection error.We use two rows to represent the optimization”
- Repeated word “due to due” (§3.1.1)
- Inconsistency in abbreviation definition (“ToF” is not explained while others like “AR” is) and use (“SSL” is used only in the abstract).

## Why use non repetitive scanning ? What is the motivation?
为什么使用非重复扫描模式

## The width distribution of Pc along Z can be described as H(|Pc_xy − 1 N ∑Pc_xy |,z) - What is H? Why not take the difference between extrema of Pc given z?
这个直接看另一个回答

## Is the basis change B necessary? Is it not possible to minimize L without using B? And B does not appear in any equation, isn’t necessary to get R/t in the correct frame of reference?
经过PCA变换的意义是什么

## It is somewhat unclear what the purpose of L1 and L2 are, please be more explicit about its function and the fact that it comes from [31]. Is it correct to say that L1 translates reflectance similarity and L2 translates closest corner similarity?
解释清除L1和L2的用处分别是什么

## Lack of limits whereas the readme of the code includes them. For instance:
- The fact that the checkerboard must be borderless
- Placement requirements (pose w.r.t. the ground and the camera; an image with “right/wrong placement” is not enough)
- Checkerboard support requirements (thin wire / thin upholder)

# Reviewer 4 (reviewer)
## 1) and 2) are also based on well-known standard techniques. 3) is based on [31], which is also presenting a method for obtaining accurate 3D corner positions from the data taken with a Lidar, so novelty and originality over [31] are also not clear.

# The accuracy of the presented method is compared with [31] and [34], which are calibration methods for ring-Lidar. The accuracy comparison presented in table 2 demonstrates better accuracy of the result with the presented method, but it is not clear why the presented method is more accurate than others. If there is a significant novel and original contribution comparing with those methods, the authors should clearly describe the contribution by pointing out the differences/advantages comparing with those methods. However, such differences/advantages are not clearly explained in this paper, but simply show the compared results in the experiment.

为什么相比于别的方法效果好，这个没有具体说

我已经在4.3明确说明了