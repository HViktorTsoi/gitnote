# A Fast Edge Extraction Method for Mobile Lidar Point Clouds

- Line segment extraction for large scale unorganized point clouds
Lin，把3D点云投影成不同视角的2D点云，然后用LSD提取线特征，再把线重投影回3D用region growing链接在一起

- Road curb extraction from mobile lidar point clouds
Xu， 用3D sobel算子，在voxel之后的点云中找候选线(任务是分割路牙)，然后利用点集的局部方向把线连在一起

- Automated registration of dense terrestrial laser- scanning point clouds using curves
用局部曲率来计算边缘， 这个适用于密集高质量点云(但是LOAM也用了？？)

- Contour detection in unstructured 3D point clouds
