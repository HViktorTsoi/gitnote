1. 结合大量信息 补全点云数据(3d GAN?)
2. 1 在object区域加attention，专注refine这些区域
夜间 过曝光 相机镜头污染

WGAN的loss无法直接优化，只能先转化为sup形式，这样必须满足1-李普希兹连续约束

discriminator 加attention

测试时 不同的前视投影生成不同的视角的图像

structure loss， 保留结构

可见光场景重建

texture loss 反射率和颜色的对应关系

related works 可行性说明

扩张视角

query x key
第i个点 最应该关注第j个点
value j个点的值(关注度是多少)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/09/22/1569160167727-1569160167740.png)


训练1HZ, 测试10Hz
夜景+路口

可微分上采样

FOV

对不同雷达的intensity分布进行重新调整

着重上采样层设计 可微分的上采样层

前视训练 全视角生成 说明具有泛化性能

利用点云分割结果 

点云特征？

解决图像截断的问题

Unet 感受野小的问题 加入膨胀卷积 和更多的skip connection

训练local和global 效率低 local无法直接使用global的结果 相当于重新训练

POINTNET 投影到前视图

stage1 补全上方点云
stage next 还原可见光图像

深度图 用log 近处的差距大 远处的差距小

集成插值+插值速度和精度 二维对比

可微分idw

光流约束

地面不平均阴影

lidar super resolution

通过修改反射率来控制样式

首先先生成有点云的部分 再补全

点云做分割 移除物体验证生成能力

加入原始点云的l w h信息

结果 车道线检测 分割 目标检测

testing比较多 用testing

** 训练前期阴影少；

object-level refine
ground-truth attention 

测评： Object-crop 计算FID；直接在生成图像上做OD/SEGMENTATION；PSNR；SSIM？
FID 全幅+object-level

Feature Loss抖动大 使用不同的学习率

不考虑颜色和材质 只考虑结构 颜色尽量均匀
（同一个patch中 只需要颜色相对亮度一致即可？ 不需要颜色绝对值相等？）
仅使用高层特征来做Feature matching

太阳方位角

prograssive discriminator

stage-2 refine

discriminator 判别调音的音效好坏

图像光流和点云光流一致性 flowmatching

3dconv Temporal domain

手标的 雷达采集的

点云分割某个目标之后再进行重建

做两部分光流的实验

museGAN-emotion

不同深度拼接

mmap分支和previsou图像分支用不同的网络结构

deep feature projection

点云分支数据增强 先random rotation 然后在特征投影之前再旋转回来

直接投影 丢失特征(如高度等)

考虑加入聚类

***2d图像重投影到3d点云 投影误差

相机死角 雷达生成

激光雷达+图像 去阴影

KITTI 过曝

feature matching loss 取不同的gamma值 选loss最小的那个进行判别

稠密投影 目标检测

普通提特征是抓住主要问题 GAN提特征是要抓住细节；使用pointnet更好的提取边界等特征；语义FM loss,是为了保证结构不变 而由于雷达缺少语义信息材质等可以通过生成来更好的表达;通常需要标定 如果发生位移相对 会对安全性造成很大的影响

不同视角生成

LAB空间或者YUV中 某一个通道对应的是反射率，可不可以从这个角度考虑，直接将LiDAR反射率和图像反射率建立关系？

类似Google net ，中继监督

×××× 反投影插值融合： 雷达投影到图像上，根据图像的纹理，插值，反向投影回雷达

×××× 雷达的遮挡和图像的遮挡不一致：图像的遮挡是由太阳光和环境光源产生的，雷达的遮挡是自身射线的光源产生的


XXX 3D Deep Snake(Deep FIshing Net)

deep snake,可以单独用扩充的关键点去训练snake head,而不用单纯依赖detection的结果,这样可以抽离出snake模块(但是要解决的问题是,snake需要输入的feature map,这个也必须由detection的backbone来提供)

先predict出bb，再回归pixel-wise mask？

PSMNet的cost volume aggregation 用cuda重新实现一遍

pointnet2 中的子pointnet实际上是去学习bacth × n_neighbor个局部点云的模式

考虑并行复杂度

feature map融合时相加和concat:如果feature是同质的或者相近的,可以用相加;否则适合用concat

iterative refine是目前在不同网络里用的比较多的方法

eye tracking label

光流和半监督结合

tsne可视化投影后的featuremap

****** 标定的时候,实在不行就做一个手工标定,手工拖动LiDAR点云,直到完全匹配上

第一个多物体 带有context info的LiDAR-image转换模型

将反射率连接到Decoder前端

introduction介绍自动驾驶

************** pointnet++实际上没有考虑进一步聚合的local特征,而仅仅是用unit pointnet单独提取每个点的特征;如果换成排了序之后的邻居节点,就可以使用鲁棒性更强的1D conv了, 保证每次都是从最近conv到最远

*** 与图像GAN不同 我们的Conditional input是从LiDAR来的 不收光源影响 鲁棒性极强
*** 不同的stride(dilation)上进行1D UP conv, 这样就能避免生成一条密集的线
*** 投影间隔这里 应该从投影之后的dead zone反推投影前缺少的点

pcl中有octree based pc change detector

LiDAR 一根线一根线的删除 然后训练网络去补全 这样可以达到自监督的效果


************ 对于深度图来说 在物体运动过程中 某个object的深度是一直发生变化的(可以观察一个sequence的video) 而不像图像的纹理 这就导致直接在图像中融合深度数据的追踪是不可行的

IDW 计算w时dist的幂次越高 远处点的影响越小

occulation统计，也就是说绝大部分像素上只有10次以下的z-冲突，因此考虑可以维护一个固定大小为H x W x 10的z-buffer-voxel(类似队列), 在做近邻搜索的时候在这个voxel中进行邻域查找即可，不需要KD-Tree。这个方法可能在GPU上效率更高，因为可以做到完全并行，而且这个方法应该在图形学里已经实现了。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/06/21/1592739372543-1592739372566.png)
但是实际上大于10的点里也有一些关键信息，下图中亮点是大于10的，可以看到一些比较细的灯杆 还有大部分远点的遮挡都很严重
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/06/21/1592740245777-1592740245777.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/06/21/1592740219140-1592740219141.png)

在2d平面上进行KD搜索时，发现大部分近邻点都落在中心点半径为10像素的范围内
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/06/21/1592740963766-1592740963767.png)

上采样生成label LiCAM生成图像 做data augmentation

*** 上边说的可以用3D voxel + 3d Conv + point Kernel实现

*** 方法中可以用膨胀-腐蚀来去除掉feature projection中的孔洞

***** RandLA的FLANN，用的是c++ subsampling，尝试用cuda重写

LiCAM BatchSize设置成小于12 越小越好

****** Interpolation 投影的时候，找到最近的K个point，由于point前后会有遮挡，需要选择固定数量的点，这个就可以用一个FC Layer的weight，与得到的点的几何特征相乘，得到加权之后的feature，可以称之attentive occulation handling module

深度补全，不需要那么精准的深度？提出新的评估标准

点云平均值叠加?

可微Min-Cut Based Segmentation of Point Clouds

验证方法：落入棋盘格边缘点和棋盘格面积之比

不同传感器点云特征分布不同 迁移学习

***** 多个LiDAR标定，直接求解棋盘格之间的刚体变换矩阵；
和相机标定结果融合，获得更准确的标定结果

pca查找点云主方向，找到实际的Z轴方向

人带着棋盘格走，光流追踪点云

** 所有参数，包括棋盘格划分的参数 都可以尝试加入到更大的优化算法中

*** 对于Human in the loop，只需要点一个seed，就可以自己完成segmentation

棋盘格摆放位置实验

*** 点云到达顺序，序列网络

二维码 标定

根据初始策略反推网络初始参数，然后再以初始参数+trust region约束为基础去优化

超大规模场景重建：V2X，路侧点云分割