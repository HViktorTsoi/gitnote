https://medium.com/@jonathan_hui/rl-natural-policy-gradient-actor-critic-using-kronecker-factored-trust-region-acktr-58f3798a4a93
trust region 优化: 是优化lower bound 函数的过程
自然梯度是从Trust region推导过来的
满足Trust region的策略梯度,推导出最终参数的梯度就是自然梯度



# 稀疏解
对稀疏解的一个直观理解是，在使得Loss最小的情况下， 尽可能的让更多的$W$的L1 Norm接近0，即获得一个稀疏的$W$。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/05/23/1590168850361-1590168850384.png)

# 点云法线
对邻居点做PCA之后， 特征值最小对应的那个基(在U中的最后一列)对应着原始点集方差最小的方向，也就是法线平面的反向，所有点基本都靠在这个平面上了。

# CMAKE pybind
CMAKE中 如果想要使用指定版本的python对应的pybind11，需要将对应版本python的inclue和lib都引入进来，如下例子所示，其中PYTHON_ROOT是python对应的environment的根目录，如~/anaconda/env/py36，PYTHON_VERSION是python的版本,如3.6
```
# python and pybind11
set(PYTHON_INCLUDE_DIRS "${PYTHON_ROOT}/include/;${PYTHON_ROOT}/include/python${PYTHON_VERSION}m")
set(PYTHON_LIBARIES "${PYTHON_ROOT}/lib/")

message("PYTHON INCLUDE DIRS: " ${PYTHON_INCLUDE_DIRS})
message("PYTHON LIBRARY: " ${PYTHON_LIBARIES})
include_directories(${PYTHON_INCLUDE_DIRS})

## compile mls upsample library
add_library(upsample_ext SHARED upsample.cpp upsample.hpp)
set_target_properties(upsample_ext PROPERTIES PREFIX "")
#target_link_libraries(upsample_ext pybind11::module ${PCL_LIBRARIES})
target_link_libraries(upsample_ext ${PYTHON_LIBARIES} ${PCL_LIBRARIES})
```

# 分组卷积
原始卷积： HxWxC1的fm，经过C2个KxKxC1的卷积核，得到HxWxC2的fm
分组卷积： 将输入按照C1划分为g组，经过C2个KxKx(C1/g)的卷积核， 同样得到得到HxWxC2的fm，注意这里，对于输入，每一组有(C1/g)个通道，而每一组输入对应着(C2/g)个卷积核，每一组输出得到的是HxWx(C2/g)的fm；

这样，卷积参数从C2xC1XKxK减小到了C2x(C/g)xKxK，减小到原来的g倍。	

而对于深度可分离卷积，其实就是：
1. 先用C1=C2=g的分组卷积，称之为深度卷积；
2. 再用1x1卷积，称之为逐点卷积；

# 一个粗暴的理解
- 公式表征方法：列方程；　
- 优化算法：解方程

# 梯度下降算法
梯度方向，就是函数值下降最快的方向，朝着梯度方向走就会使cost函数值下降。

引入二阶梯度的目的是，不仅直到函数值在某个方向是是不是上升/下降的，而且知道上升/下降的变化趋势，并可以适应性的调整梯度大小。例如在接近谷底的地方，根据二阶导数知道，梯度越来越缓，此时引入二阶梯度让参数更新更小的值，可以防止越过最优点；而在半山上，通过二阶梯度知道梯度还很大，此时引入二阶梯度更新更大的值，可以更快的到达谷底。

# 用PCA求出来的基底变换矩阵，可以任意调整三个底的顺序和数值，然后左乘到原始点云上即可得到新底下的点集； 即可以自己写一个新的基底矩阵(从I开始)进行变换

# 右手法则
123指头分别对应123轴，即
轴-手指-颜色
- X-大拇指-R
- Y-食指-G
- Z-中指-B

# PCL出错状态机:
在特定版本的eigen上装了pcl之后，再安装其他版本的eigen，大概率pcl中的很多module都不可用了

# 刚体变换的表现形式，与DL中参数和输入的线性组合方式非常像, R=W, t=b

# 将传统方法和深度学习融合, 其实就是传统方法中可学习/需要优化的参数连到NN中, 和普通的NN参数一起优化，即可以看成一个更大的更宏观的优化方法，求解最优解问题。在此过程中，只需要保证在梯度传播路径中，传统方法接入点的传播路径与NN的前后传播路径是导通的即可。

# 图优化建图方式
节点：优化变量
边：误差项

例子：节点1：相机位姿；节点2～N：特征点3D坐标；边1~N-1:从3D坐标估计值投影到相机位姿的成像平面后，与实际测量值的loss
BA就是同时调整相机位姿和观测得到的特征点3D坐标

pointcloud 描述子

用objectdetection的思路做global localization

# LiDAR Super Resolution
改进点：loss，网络，序列信息
效果较好：TCN，比原来要高8.几%

# 怎么鼓吹自己
1. 介绍方向重要
2. 自己做的方向很难
3. 自己做了什么，只说思路，不要说流程
4. 要比同行做得好，引用多少多少
5. 有什么成果

# 关于目标检测的scale
在2D图像中，因为投影的问题，scale很重要；
而在3D点云中，scale就没那么重要，因为同样的物体在不同距离大小是相同的。但是有另一个问题需要考虑，就是稀疏程度不同。

# 点云有光流吗？
光流描述的是图像中的像素运动，是通过光照产生的灰度不变假设计算出来的；对于点云来说，这样的假设在什么量上成立？

# KF
1. 卡尔曼滤波的预测过程，实际上就是用运动方程，通过上一个时刻的状态来预测当前时刻的状态，(比如已知上一时刻的位姿,已知运动为匀速运动，那么就可以预测下一时刻的位姿);
2. 更新过程，其实就是用bayes公式，以及观测方程(比如在某一个位姿上对路标点进行观测)，来计算状态的后验概率分布(即当前时刻的最终状态);
3. 对于EKF，是将运动方程和观测方程进行线性化(即一阶Taylor展开)，形式上就是将运动和观测方程的线性做乘换成函数形式，然后在运动方程构成的先验分布协方差上加一个一阶导的二次型。然后再按照1,2的步骤进行预测和更新。

# 边缘化
对于多变量的联合概率分布，例如X和Y，求单个变量的概率分布(称为边缘分布)，比如P(X)或者P(Y)的过程，叫做边缘化，这个过程中其实就是将其他不相关变量的概率求和(积分),其他变量就叫做被边缘化的量。比如求P(X)，那么就是将P(X|Y)对Y求积分。
在slam后端优化时，边缘化是指，因为在对H阵进行Schur消元后，是先单独求解出所有的相机位姿Xc，再求解路标Xp，因此从概率的角度来讲，就是先确定Xc的边缘分布，而将Xp进行边缘化。

在边缘化掉某个位姿变量之后,对新的H阵引入的稠密性,通过可视化可以看出来,其代表了剩下的位姿和路标点之间建立的新的联系,即边缘化位姿之后保留了剩下的路标和位姿之间的约束关系;将某个变量margin掉之后,就可以不用管它,然后先求解其他变量

# 在位姿图中，已知量只有每两个相机之间的相对位姿(用特征匹配或者其他方法)，未知量是每个相机的绝对位姿。

有一个关键问题：没有回环的时候怎么进行图优化？这个时候所有帧的相对位姿仅仅构成一个链，而不是图。

第一种思路:这时候可以考虑用这样的方法：除了在每帧之前用特征匹配计算相对位姿之外，还要对相互间隔不远的两个关键帧计算相对位姿，这样就会构成有约束的图了，图的边就有两种，一种是相邻帧的相对位姿，另一种是间隔了几帧的关键帧相对位姿，因为这两种位姿是独立计算出来的，因此构成图之后就会有误差，就有优化的空间。

第二种思路:
图的节点是关键帧的绝对位姿，其初始值是仅仅经过每一帧VO叠加之后得到的绝对位姿；
图的边(测量值)是两个关键帧之间经过局部优化(例如BA)之后得到的更准确的相对位姿；
这样图的节点之间和观测值就产生了误差，有优化的空间。

这么看起来，有路标点和位姿的BA在没有回环的时候就更像图优化；而没有回环的位姿图其实不构成图。

# g2o里,VertexSE3EXPMAP跟Edge传递的增量是6D的李代数向量；但是在内部更新变量和实际内部存储的时候，都是用的一个3D trans+4元数来做的。（这样设计可能是考虑优化时李代数直观，实现时四元数方便？）

# g2o里，在对Vertex的优化变量进行增量更新时，要注意 增量 和 估计值的乘法是不满足交换律的，一般要左乘增量；

# 坐标系A到坐标系B的正交转换
尝试从A旋转到B,在旋转的过程中看每一步绕哪个轴转了多少度,最后就可以用transforms3D中的eular2mat计算出从系A到系B的旋转矩阵R,这个矩阵左乘原向量就得到新坐标系下的向量.

# UTM坐标系就是我们通常所说的平面直角坐标系

# Cartographer要在install_isolated的launch和config里边更改配置文件,如果想离线建图,要使用offline node

# PYBIND11到底怎么玩
如果没有find_package(PCL) 完全按照pybind11的文档

如果find_package(PCL) 必须加上
include_directories(/home/hyx/miniconda3/envs/torch/include/python3.6m)
其中include_directories中的参数是你希望编译到的目标python版本的include目录
可以通过
python -c "from sysconfig import get_paths;print(get_paths()['include'])"
查到


# transformer in CV
transformer最大的优势是可以并行处理一个序列（用attention），而不像RNN之类的必须要先处理一个词再接着下一个

transformer在分类上结果，小数据集没有cnn好，但是大数据集预训练之后效果比部分cnn好得多

可能的原因：
1. CNN是参数共享，transformer里有很多fc的，参数量多一些，数据量更大可能会更好一点；
2. 全attention结构确实比CNN能够更好的建模全局信息；

# 双目也并不都是用来估计稠密深度的， 可以在求出来匹配的特征点之后三角化出绝对的深度

# 3d slam提特征与上采样的关系
目前来看，提特征跟点云密度关系其实不大，如果想要对点云进行上采样的话，重点是把对应有效特征的那部分点上采样出来

# 生成图像质量评价标准
1. Inception Score
2. FID, Frechet Inception Distance score
3. SSIM
4. 借助分类任务进行评估
5. subjective scores

