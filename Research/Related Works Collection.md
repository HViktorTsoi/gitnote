# A Fast Edge Extraction Method for Mobile Lidar Point Clouds

- Line segment extraction for large scale unorganized point clouds
Lin，把3D点云投影成不同视角的2D点云，然后用LSD提取线特征，再把线重投影回3D用region growing链接在一起

- Road curb extraction from mobile lidar point clouds
Xu， 用3D sobel算子，在voxel之后的点云中找候选线(任务是分割路牙)，然后利用点集的局部方向把线连在一起

- Automated registration of dense terrestrial laser- scanning point clouds using curves
用局部曲率来计算边缘， 这个适用于密集高质量点云(但是LOAM也用了？？)

- Contour detection in unstructured 3D point clouds
用法线、特征值等作为feature， 做有监督的分类，把edge区分出来。但是没有训练数据就不工作了

- Vision-based localization using an edge map extracted from 3d laser range data
把边缘看做面的交界，先做平面分割，然后用平面交线作为edge。这个方法在小平面处不工作。

- Edge detection and feature line tracing in 3D-point clouds by analyzing geometric properties of neighborhoods
把点投影到局部平面上， 根据angular gap(一种度量，论文里有详细解释)，来判断哪些点是edge

# BALM
- A correlation-based approach to robust point set registration
使用互相关，可以实现多帧的配准；但是需要计算所有对之间的互相关，比较耗时；

- Real-time correlative scan matching
互相关配准的实时实现；

- Lost in translation (and rotation): Rapid extrinsic calibration for 2d and 3d lidars
GPU实现的互相关配准

- Eigen-factors: Plane estimation for multi-frame and time- continuous point cloud alignment
在多帧之间利用共同观测到的平面来优化位姿序列

- Tightly coupled 3d lidar inertial odometry and mapping
将当前帧与滑动窗口内之前已经构建的map进行配准；但是和BA相比，忽略了滑窗内部各个帧的相互关系

- Efficient continuous-time slam for 3d lidar-based online mapping
使用多尺度的栅格地图，可以实现multi-view配准；但是太占用运算资源和内存；

# Loam Livox 回环检测
- Place recognition using keypoint voting in large 3d lidar datasets
- Structure-based vision-laser matching
这两篇类似，从点云中提keypoint，然后用Gestalt descriptors描述子， 每个keypoint给他附近的点(可能是目标帧的)vote，最后的评分作为判断是否回环的score；

- Appearance-based loop detection from 3d laser data using the normal distributions transform
直接用NDT的描述子来计算两帧的相似度

- Segmatch: Segment based place recognition in 3d point clouds
通过场景中的语义信息(tree， building)等进行matching

- Pointnetvlad: Deep point cloud based retrieval for large-scale place recognition
- Netvlad: Cnn architecture for weakly supervised place recognition
用pointnet和netvlad结合起来，通过网络拿到场景点云的embedding，来做回环的描述子

- Learning to see by moving
从ego motion估计任务中学习VO要用的feature

- Deepvo: Towards end-to-end visual odometry with deep recurrent convolutional neural networks
使用RNN从video中学习odometry

- Demon: Depth and motion network for learning monocular stereo
- Deeptam: Deep tracking and mapping
从单目中学习深度，做SFM任务

- Eng: End-to-end neural geometry for robust depth and pose estimation using cnns
利用DNN来预测深度、光流，从而估计相机运动

- Unsupervised learning of depth and ego-motion from video
SFM-learner，用自监督的方式学习深度和motion

- Unsupervised scale-consistent depth and ego-motion learning from monocular video
解决SFM-learner的尺度不一致问题

- UnDeepVO，Depth-VO- Feat
利用双目来解决尺度不一致问题

- Geonet: Unsupervised learning of dense depth, optical flow and camera pose,
- Competitive collaboration: Joint unsupervised learning of depth, camera motion, optical flow and motion segmentation
结合光流估计、深度估计等做slam

- CNN-SLAM
CNN做单目深度估计

- CNN-SVO
利用CNN来初始化feature点处的深度，来解决初始map的深度不确定性

- Deep virtual stereo odometry: Leveraging deep depth prediction for monocular direct sparse odometry
depth估计加入到DSO中

- Pose graph optimization for unsupervised monocular visual odometry
加入了位姿图优化
