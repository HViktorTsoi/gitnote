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
1. 卡尔曼滤波的预测过程，实际上就是用运动方程，通过上一个时刻的状态来预测下一时刻的状态，(比如已知上一时刻的位姿,已知运动为匀速运动，那么)
