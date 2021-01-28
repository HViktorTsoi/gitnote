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


# GAN生成图像质量评价标准

## 1. Inception Score
将生成图像送进inception, 得到1x1000的预测向量; 

	1. 如果生成的样本比较真实,那么Inception应该给出某个比较明确的类别, 向量中某个值比较高, 因此预测向量的信息熵越低越好;

	2. 如果生成的样本多样性好,那么对于产生的一批样本,其对应的这批类别标签的分布应该比较均匀(各个类别都生成了一些), 因此类别标签的信息熵越高越好;

	3. 最后用样本和标签的互信息来计算Inception Score, 互信息的计算方法就是求2-1的差值; 等价于标签分布和标签-样本条件分布的KL散度;

	4. 缺陷: 

		1. GAN过拟合时,如果生成的完全都是训练集的真实图像一样的图像, 那么Inception score也会很低; 

		2. 偏爱ImageNet中的物体类别，而不是注重真实性。GAN生成的图片无论如何逼真，只要它的类别不存在于ImageNet中，IS也会比较低;

		3. 若GAN生成类别的多样性足够，但是类内发生模式崩溃问题，IS无法探测;

		4. IS只考虑生成器的分布, 而忽略数据集的分布;

		5. IS的高低会受到图像像素的影响


## 2. FID, Frechet Inception Distance score

	1. 分别把生成器生成的样本和判别器生成的样本送到分类器中, 抽取分类器的中间层的抽象特征, 假设该抽象特征符合多元高斯分布;

	2. 估计生成样本和真实样本产生的特征的均值和方差, 这样就拿到了生成和真实这两个分布的参数;

	3. 计算这两个分布的Fréchet距离, 生成图像质量的度量;
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/07/1610014718100-1610014718101.png)


## 3. MMD, Maximum Mean Discrepancy

	1. 找一个核函数k将两个样本映射为一个实数,这个核函数需要能度量两个样本的相似程度,越相似函数值越高;

	2. 用所有的样本来计算MMD距离, 其可以来度量生成分布和真实数据集分布的相似程度;

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/07/1610014680718-1610014680719.png)


## 4. Wasserstein Distance

	1. 首先要有一个训练好的判别器D;

	2. 对生成图像和训练集图像, 分别用判别器来计算出判别值;

	3. 然后用所有的判别值来计算Wasserstein距离;

	4. 这个评价指标可以探测到生成样本的简单记忆情况和模式崩溃情况, 但是因为D是在特定数据集上训练出来的, 只能用来评价这个数据集上的图像, 比如在苹果图像上训练的,就不能用来判别橘子并算Wasserstein距离;


## 5. one-Nearest Neighbor Classifier

	1. 把生成的图像打上0标签, 真实图像打上1标签, 混合在一起构成数据集D(共2n张图像);

	2. 每次从D中拿出2n-1个样本,训练一个1-NN(k=1的KNN)分类器;

	3. 用剩下的1个样本做测试; 如此重复留一法验证2n次,直到所有样本都被验证一遍,计算最终的准确率;

	4. 如果准确率接近0%, 说明出现简单记忆现象: 每个生成样本都和真实样本距离很近, 在1-NN查找的时候, 永远只能找到和留一的生成(真实)样本对应那个真实(生成)样本; 
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/07/1610014530089-1610014530089.png)
	
	5. 如果准确率接近100%, 说明生成的样本分布和真实样本分布差的很远,很容易就能够区分开,说明生成的质量很差; 
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/07/1610014539557-1610014539558.png)

	6. 当准确率接近50%的时候, 才说明生成和真实样本的距离接近, 而又不是过拟合导致很近;  
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/07/1610014435334-1610014435335.png) 
	

## 6. NRDS, Normalized Relative Discriminative Score

可用于多个GAN模型的比较, 想法是：实践中，对于训练数据集和GAN生成器生成的样本集，只要使用足够多的epoch，总可以训练得到一个分类器C，可以将两类样本完全分开，使得对训练数据集的样本，分类器输出趋于1，对GAN生成的样本，分类器输出趋于0。

	1. 在每个epoch中, 对n个GAN生成的样本打上标签0, 对真实样本打上标签1, 合成一批训练集;

	2. 用合成的数据集训练模型一个epoch; 

	3. 然后当前训练的模型对n个GAN的数据进行测试, 记录输出(0~1的值, 越接近1说明分类器认为这个图像越像真实样本)和当前的epoch;

	4. 这样经过多个epoch, 对于每个GAN的数据, 都能绘制一条 epoch-输出 曲线, 这条曲线面积越大, 说明在分类器用了越少的epoch来认为某个GAN的样本是真实图像, 这就能说明生成图像的质量更高;  

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/07/1610014399416-1610014399443.png)
	
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/07/1610014370518-1610014370620.png)


## 7. GANtrain-GANtest

是一种评价流程, 定义真实图像训练样本集St, 真实图像验证集Sv, 生成图像样本集合Sg; 

	1. 在真实图像St上训练分类器, 并在真实验证集Sv上测试, 准确率记为GANBase;

	2. 在生成图像上Sg训练分类器, 并在真实验证集Sv上测试, 准确率记为GANtrain;

	3. 在真实图像St上训练分类器, 并在生成图像Sg上测试, 准确率记为GANtest;

	4. 比较GANBase和GANtrain, train较小, 说明GAN存在问题, 可能是发生了模式丢失, 或者样本不够真实, 让分类器学不到有效的特征; 或者生成的类别分的不够开; 如果二者比较接近, 说明生成图像的质量较高, 和训练集有很多模式相似性;

	5. 比较GANBase和GANtest, 如果test较高, 说明发生了过拟合, 发生了简单记忆问题; 如果test较低, 说明生成的数据分布不好, 图像质量不高;  


## 8. SSIM, PSNR, SD

这一系列是对图像质量进行评估, 主要是来评估图像的锐度/对比度/亮度分布, 而不是基于图像内容进行评价; 
适合任务:超分辨, 压缩后重建等 GT和预测值 以及 输入和输出值 有较明确的映射关系的任务

- SSIM, 结构相似度, 基本思路是基于图像局部灰度的均值方差,来计算亮度差异,对比度差异,以及结构差异,算法参考WIKI;

- PSNR, 峰值信噪比, 基本思路是求两幅图像逐像素差的倒数, 算法参考WIKI;

- SD, 锐度差异, 算法类似PSNR, 但是比较的是两幅图像的锐度差;


## 9. Perceptual loss

类似vid2vid中的feature matching loss, 用某种Encoder, 对真实图像和生成图像分别进行encode, 然后对输出的feature进行比较;


## 10. 借助分类/检测/分割任务进行评估

在生成图像上进行分类/检测/分割任务, 本质上和Perceptual loss, 以及GANtrain-GANtest中的Base-test比较类似,


## 11. Forward / Backward Consistency

这个是WC vid2vid针对他的模型提出的, 来判断视频生成任务中, 生成的视频是否有时间上的一致性: 

	1. 对于一个时长为t的视频序列的input, 将其顺序-逆序拼接在一起, 构成时长为2t的video input;

	2. 将video input作为输入, 送进模型中, 得到预测的视频输出;

	3. 比较第0帧和最后一帧的图像的差异(论文中用的就是逐像素差); 如果模型有时间一致性, 那么第0帧和最后一帧应该是完全一致的(同理第1和第n-1, 第2和第n-2等也应该是一致的);


## 12. subjective scores

人类评估, 目前已知的工具有Amazon Mechanical Turk (AMT), 在测试中, 随机 同时 给两个video(图片), 不告诉被测试者是什么模型生成的, 从几个维度上评价, 选择出哪个结果更真实;


## 13. Synthetic-Neuroscore

参考论文Synthetic-Neuroscore: Using a neuro-AI interface for evaluating generative adversarial networks

让人类在看GAN生成图像和真实图像的时候, 分别带上EEG设备采集脑电信号, 然后通过对EEG进行分类和对比, 来确定生成的图像是否和真实图像一致;



# torch里的DataParallel是把batchsize平均分到多个gpu上，这里要看一下跨卡batch norm

# 局部坐标系的点云转换到世界坐标系
对于局部坐标系(LiDAR为中心的点云)Xl, 现在有其在世界坐标系下，以起点为原点，当前位置为终点的的里程计P(w->l),或者称之为当前位姿，那么这一帧点云在世界坐标系下的坐标Xw就是

Xw = P(w->l) Xl



# squeezenet训练

注意图像分辨率小的时候，不要进行过多的降采样，否则会使信息丢失

ATLAS的AIPP在读取图片的时候，要注意RBG通道的顺序要和网络训练的顺序一致，并且图像的归一化/缩放参数也要和网络训练的一致。



# 坐标系转化原则
从A坐标系转换到B坐标系，要用B->A的转换，
