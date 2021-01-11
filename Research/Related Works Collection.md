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
从点云中提keypoint，然后用
