# carla v2x数据集生成
1. LiDAR的intensity, 用RGB转换成grey之后索引, 或者融合semantic标注, 对车道线路面等赋予一些特殊的intensity值
2. 对label 进行反射率纹理映射
3. rsu要放在什么位置, 这个最好能够自动实现部署, 或者至少可以人手工指定一些位置
4. 怎么实现车辆沿着一定的waypoint经过指定的rsu. 参考https://github.com/carla-simulator/carla/issues/3890, 以及carla的PythonAPI/agent实现
5. 怎么获取相机的内参, 怎么获取vehicle-cam到rsu-lidar之间的tf

# opensfm
尝试在Py27安装

# 一个观察到的现象: 以灯杆作为reference, 由于重建的点质量较差, 配准效果差; 而如果以重建作为reference, 配准的效果就更好

# GPS优化崩溃, 有以下报错:  Path 'GloablPath' contains unnormalized quaternions. This warning will on
# 在外参准确的情况下, GPS优化会崩溃/跳变, 猜测可能是GPS方差设置的太小(0.06), 导致relax的限度太低, 系统只能通过位姿跳变的方式消化多余的误差; 将方差设置为0.5之后, 效果平滑了很多; 
# 是不是时间同步的问题???? 找不到对应帧了, 那一帧VIO就没有约束, 结果导致大量优化误差都relax到那一帧位姿上

#　另外需要确定是不是因为GPS频率过高导致的问题
确实有影响, GPS1HZ, 方差0.4的时候就不会崩溃
# 考虑用地面上的竖直杆子作为约束

# Relocalization思路
1. 跑VIO, 将现有的需要地图优化的Keyframe Odometry,以及对应的image, /vins_estimator/keyframe_point
2. 注意/vins_estimator/keyframe_point里的point3d是VIO世界坐标系,如果经过gtsamleKF的位姿,那么point3d对应的点坐标也应该优化,保持一致
3. 保存VINS格式的pose graph, 其中VIO和PG都是优化得到的POSE, LOOP INFO全部设置为0(LOOP INFO存的是之前回环优化之后的pose和原始VIO之间的相对值, 格式为x,y,z,qw,qx,qy,qz,yaw); loop index是当前帧所回环到的历史帧ID, 默认都设置成-1
4. 保存KF对应的keypoints以及descriptors, 注意分为两部分, 一个是VINS给的, 一个是LoopFusion自己重新检测的. 为了获得这些信息,需要过一遍KeyFrames的构造函数
5. 1~4的过程可以考虑将优化之后的Pose以及更新的keyfram_point, image作为伪VINS的输出给loop_fusion, 然后使用loop_fusion的保存功能将pose graph存下李
6. 在定位过程中, 首先载入保存的posegraph； 然后等待loop fusion将当前帧的与先验地图进行loop, 来做全局定位； 这里可能需要关掉optimization（如果只要最新的localization结果）


\textbf{Infrastructure-based Localization and Mapping.}

 Recent works \cite{zhao2021heterogeneous, zhu2020voc, lu2021carom, luo2021empirical, hua2018vehicle} focus on object-level localization and re-identification from road-side cameras or LiDARs. 
 
 \cite{zhu2020voc} proposed Voc-ReID,fusing the triplet vehicle-orientation-camera information and reforming background/shape similarity as camera and orientation re-identification. 
 
 CAROM \cite{lu2021carom} presented a vehicle tracking and traffic scene reconstruction framework utilizing infrastructure cameras, which processes traffic monitoring videos and converts them to anonymous data structures of vehicle type, 3D shape, position, and velocity for traffic scene reconstruction and replay.
 
 \cite{hua2018vehicle} designed a vehicle detection and speed estimation methods from roadside lidar, which extracts motion points from static background points, and tracks the vehicle points by utilizing geometric prior.

 These approaches require the vehicle to be in the field of view (FOV) of the roadside unit, and also it is challenging for the roadside unit to associate the detection with actual vehicles, therefore they are more suitable for traffic flow monitoring from the infrastructure-side, rather than vehicle-centralized autonomous driving scenario. 

\cite{ma2019efficient} proposed a radio-based vehicle localization method, by applying weighted linear least squares to the vehicle kinematic model as well as multiple distance measurements between the vehicle and the roadside units. \red{Describe the drawbacks of radio-based localization methods.}

To the best of our knowledge, there are few previous works on roadside infrastructure-assisted mapping.


\textbf{Visual SLAM.} 

In this work, we focus on mapping and localization primarily using visual sensors, which are typically of low prices, low power consumption, and are widely utilized in current commercial vehicles. To date, most visual maps consist of 3D distinctive points, computed by probabilistically aggregating and optimizing information from point features detected from input images. 

ORB-SLAM3 \cite{campos2021orb} is a feature-based tightly-integrated SLAM system that fully relies on Maximum-a-Posteriori (MAP) estimation by reusing historical map information, and can be performed under multiple vision and inertial sensor combinations.  

RTAB-Map \cite{labbe2019rtab} proposed a real-time appearance-based mapping approach, which creates dense 3D and 2D representations of the environment online and also supports mixed-modality vision sensors.

CarMap \cite{ahmad2020carmap} proposed a multi-agent SLAM Framework that fuses map segments from multiple vehicles through crowd-sourcing map updates.

Recently a series of visual-odometry(VO) or visual-inertial odometry(VIO) \cite{usenko2019visual, qin2018vins, geneva2020openvins} approaches are also proposed, which focus on the real-timeness and accuracy of the front-end of SLAM by utilizing sliding-windows-based joint optimization methods.

Due to the degenerated sensor measurement and the error during feature tracking, the front end of SLAM algorithms suffers from inevitable accumulated errors after long-term explorations. Most approaches utilize loop closure to eliminate this error, however, in a large-scale driving scenario, the vehicles merely driving through the same roads from a similar perspective repeatedly during a short period of time, increases the failure probability of loop closure significantly, therefore the map-centralized approaches such as ORB-SLAM \cite{campos2021orb} may not work as excepted, since it heavily relies on loop closure constraint to optimize a global-consistent map. Another approach to eliminating the cumulative error is to fuse global constraints, such as GNSS\cite{kiss2019gps, qin2019general, chiang2020performance}, however, the mapping precision will thus depend on the accuracy of the GPS measurements, which is usually inferior under GPS-denied environments, such as long-tunnels or urban areas. 


\textbf{LiDAR-Camera Fusion.} 

With the recent widespread application of LiDAR, there are a series of studies on the fusion of LiDAR point clouds and camera images, especially for the perception \cite{qian2021robust,vora2020pointpainting, ouyang2022saccadefork} and localization \cite{debeunne2020review} tasks of autonomous driving. 

R2LIVE \cite{lin2021r} proposed a tightly-coupled sensor fusion framework, which fuses measurement from LiDAR, inertial sensor, and visual camera to achieve robust and accurate state estimation. The motion state within the framework is estimated by error-state iterated Kalman-filter to guarantee real-time performance and then refined with factor graph optimization. 

LVI-SAM \cite{shan2021lvi} proposed a semi-tightly-coupled lidar-visual-inertial odometry via smoothing and mapping, which improves the accuracy of the visual odometry by extracting depth information for visual features from lidar measurements, and conversely utilized the visual odometry as initial estimation for LiDAR Mapping, to improve the robustness in the geometric degenerated environment. 

\cite{zhao2021super} employs an IMU-centric fusion pipeline to bridge the information from the camera and LiDAR, which combines the advantages of loosely coupled methods with tightly coupled methods and recovers motion in a coarse-to-fine manner.

 Most of the existing fusion approaches assume that the LiDAR and camera are mounted to a fixed base frame, and also strictly require sensor measurement to share the same FOV. Such methods are hard to be applied in vehicle-infrastructure fusion scenarios, where the relative position between the vehicle and the infrastructure constantly changes, and they do not share the similar FOV in most cases. Besides, the cost of equipping integrated LiDAR-camera sensors on vehicles is still high, which also limits the mess deployment of autonomous vehicles. In this paper, we propose a new paradigm for LiDAR-camera fusion, which only requires low-cost cameras mounted on vehicle, and can achieve high-precision localization and mapping by fusing the point cloud information from roadside infrastructures.

<!-- %=====================================================================

@inproceedings{zhao2021heterogeneous,
  title={Heterogeneous Relational Complement for Vehicle Re-identification},
  author={Zhao, Jiajian and Zhao, Yifan and Li, Jia and Yan, Ke and Tian, Yonghong},
  booktitle={Proceedings of the IEEE/CVF International Conference on Computer Vision},
  pages={205--214},
  year={2021}
}

@inproceedings{zhu2020voc,
  title={VOC-ReID: Vehicle re-identification based on vehicle-orientation-camera},
  author={Zhu, Xiangyu and Luo, Zhenbo and Fu, Pei and Ji, Xiang},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops},
  pages={602--603},
  year={2020}
}

@inproceedings{luo2021empirical,
  title={An empirical study of vehicle re-identification on the AI City Challenge},
  author={Luo, Hao and Chen, Weihua and Xu, Xianzhe and Gu, Jianyang and Zhang, Yuqi and Liu, Chong and Jiang, Yiqi and He, Shuting and Wang, Fan and Li, Hao},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  pages={4095--4102},
  year={2021}
}

@inproceedings{lu2021carom,
  title={CAROM-Vehicle Localization and Traffic Scene Reconstruction from Monocular Cameras on Road Infrastructures},
  author={Lu, Duo and Jammula, Varun C and Como, Steven and Wishart, Jeffrey and Chen, Yan and Yang, Yezhou},
  booktitle={2021 IEEE International Conference on Robotics and Automation (ICRA)},
  pages={11725--11731},
  year={2021},
  organization={IEEE}
}

@inproceedings{hua2018vehicle,
  title={Vehicle tracking and speed estimation from traffic videos},
  author={Hua, Shuai and Kapoor, Manika and Anastasiu, David C},
  booktitle={Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition Workshops},
  pages={153--160},
  year={2018}
}


@article{ma2019efficient,
  title={An efficient V2X based vehicle localization using single RSU and single receiver},
  author={Ma, Sugang and Wen, Fuxi and Zhao, Xiangmo and Wang, Zhong-Min and Yang, Diange},
  journal={IEEE Access},
  volume={7},
  pages={46114--46121},
  year={2019},
  publisher={IEEE}
}


%=====================================================================
@article{campos2021orb,
  title={Orb-slam3: An accurate open-source library for visual, visual--inertial, and multimap slam},
  author={Campos, Carlos and Elvira, Richard and Rodr{\'\i}guez, Juan J G{\'o}mez and Montiel, Jos{\'e} MM and Tard{\'o}s, Juan D},
  journal={IEEE Transactions on Robotics},
  volume={37},
  number={6},
  pages={1874--1890},
  year={2021},
  publisher={IEEE}
}

@article{labbe2019rtab,
  title={RTAB-Map as an open-source lidar and visual simultaneous localization and mapping library for large-scale and long-term online operation},
  author={Labb{\'e}, Mathieu and Michaud, Fran{\c{c}}ois},
  journal={Journal of Field Robotics},
  volume={36},
  number={2},
  pages={416--446},
  year={2019},
  publisher={Wiley Online Library}
}

@inproceedings{ahmad2020carmap,
  title={$\{$CarMap$\}$: Fast 3D Feature Map Updates for Automobiles},
  author={Ahmad, Fawad and Qiu, Hang and Eells, Ray and Bai, Fan and Govindan, Ramesh},
  booktitle={17th USENIX Symposium on Networked Systems Design and Implementation (NSDI 20)},
  pages={1063--1081},
  year={2020}
}


@article{usenko2019visual,
  title={Visual-inertial mapping with non-linear factor recovery},
  author={Usenko, Vladyslav and Demmel, Nikolaus and Schubert, David and St{\"u}ckler, J{\"o}rg and Cremers, Daniel},
  journal={IEEE Robotics and Automation Letters},
  volume={5},
  number={2},
  pages={422--429},
  year={2019},
  publisher={IEEE}
}

@article{qin2018vins,
  title={Vins-mono: A robust and versatile monocular visual-inertial state estimator},
  author={Qin, Tong and Li, Peiliang and Shen, Shaojie},
  journal={IEEE Transactions on Robotics},
  volume={34},
  number={4},
  pages={1004--1020},
  year={2018},
  publisher={IEEE}
}

@inproceedings{geneva2020openvins,
  title={Openvins: A research platform for visual-inertial estimation},
  author={Geneva, Patrick and Eckenhoff, Kevin and Lee, Woosik and Yang, Yulin and Huang, Guoquan},
  booktitle={2020 IEEE International Conference on Robotics and Automation (ICRA)},
  pages={4666--4672},
  year={2020},
  organization={IEEE}
}


@article{chiang2020performance,
  title={Performance enhancement of INS/GNSS/Refreshed-SLAM integration for acceptable lane-level navigation accuracy},
  author={Chiang, Kai-Wei and Tsai, Guang-Je and Chu, Hone-Jay and El-Sheimy, Naser},
  journal={IEEE Transactions on Vehicular Technology},
  volume={69},
  number={3},
  pages={2463--2476},
  year={2020},
  publisher={IEEE}
}

@article{kiss2019gps,
  title={GPS-SLAM: an augmentation of the ORB-SLAM algorithm},
  author={Kiss-Ill{\'e}s, D{\'a}niel and Barrado, Cristina and Salam{\'\i}, Esther},
  journal={Sensors},
  volume={19},
  number={22},
  pages={4973},
  year={2019},
  publisher={Multidisciplinary Digital Publishing Institute}
}

@article{qin2019general,
  title={A general optimization-based framework for global pose estimation with multiple sensors},
  author={Qin, Tong and Cao, Shaozu and Pan, Jie and Shen, Shaojie},
  journal={arXiv preprint arXiv:1901.03642},
  year={2019}
}



%=====================================================================
@inproceedings{qian2021robust,
  title={Robust multimodal vehicle detection in foggy weather using complementary lidar and radar signals},
  author={Qian, Kun and Zhu, Shilin and Zhang, Xinyu and Li, Li Erran},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  pages={444--453},
  year={2021}
}

@inproceedings{vora2020pointpainting,
  title={Pointpainting: Sequential fusion for 3d object detection},
  author={Vora, Sourabh and Lang, Alex H and Helou, Bassam and Beijbom, Oscar},
  booktitle={Proceedings of the IEEE/CVF conference on computer vision and pattern recognition},
  pages={4604--4612},
  year={2020}
}

@article{ouyang2022saccadefork,
  title={Saccadefork: A lightweight multi-sensor fusion-based target detector},
  author={Ouyang, Zhenchao and Cui, Jiahe and Dong, Xiaoyun and Li, Yanqi and Niu, Jianwei},
  journal={Information Fusion},
  volume={77},
  pages={172--183},
  year={2022},
  publisher={Elsevier}
}

@article{debeunne2020review,
  title={A review of visual-LiDAR fusion based simultaneous localization and mapping},
  author={Debeunne, C{\'e}sar and Vivet, Damien},
  journal={Sensors},
  volume={20},
  number={7},
  pages={2068},
  year={2020},
  publisher={Multidisciplinary Digital Publishing Institute}
}

@inproceedings{rashed2019fusemodnet,
  title={Fusemodnet: Real-time camera and lidar based moving object detection for robust low-light autonomous driving},
  author={Rashed, Hazem and Ramzy, Mohamed and Vaquero, Victor and El Sallab, Ahmad and Sistu, Ganesh and Yogamani, Senthil},
  booktitle={Proceedings of the IEEE/CVF International Conference on Computer Vision Workshops},
  pages={0--0},
  year={2019}
}


@inproceedings{zhao2021super,
  title={Super odometry: IMU-centric LiDAR-visual-inertial estimator for challenging environments},
  author={Zhao, Shibo and Zhang, Hengrui and Wang, Peng and Nogueira, Lucas and Scherer, Sebastian},
  booktitle={2021 IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)},
  pages={8729--8736},
  year={2021},
  organization={IEEE}
}

@article{lin2021r,
  title={R $\^{} 2$ LIVE: A Robust, Real-Time, LiDAR-Inertial-Visual Tightly-Coupled State Estimator and Mapping},
  author={Lin, Jiarong and Zheng, Chunran and Xu, Wei and Zhang, Fu},
  journal={IEEE Robotics and Automation Letters},
  volume={6},
  number={4},
  pages={7469--7476},
  year={2021},
  publisher={IEEE}
}

@inproceedings{shan2021lvi,
  title={Lvi-sam: Tightly-coupled lidar-visual-inertial odometry via smoothing and mapping},
  author={Shan, Tixiao and Englot, Brendan and Ratti, Carlo and Rus, Daniela},
  booktitle={2021 IEEE International Conference on Robotics and Automation (ICRA)},
  pages={5692--5698},
  year={2021},
  organization={IEEE}
} -->

Background Filtering

Inspired By [], we propose a background filtering pipeline

对比实验, 把全局的轨迹误差和靠近RSU的轨迹误差放在一起对比

scalability, 因为计算是在车端, 并且所有车端接收到的数据都是一样的, RSU只需要隔一段时间记录自己周围的环境变化即可； 如果RSU部署的多, 车辆相当于得到了近似实时的高精地图

notice that the driving direction there是相反的, 在这里仅仅靠图像是无法形成回环约束的

use only Infrastructure


Overview

The design objective of VILM is to achieve high precision visual mapping and localization with the aid of the 3D measurement from the roadside infrastructure. As shown in Fig. \ref{}, we first utilize the visual odometry and submap reconstruction to recover the 3D geometric information of the road environment, and also to obtain a accumulive estimation of the vehicle pose. While driving through the roadside infracture, the vehicle receives the static scene point cloud extracted from the infracture. Then, with the proposed elastic registration approach, the localization of the visual submap with respect to the infracture frame is estimated. We then design a factor graph-based method to fuse the registration resualt with the visual odmmetry, and a global consistent 3D map can be obtained after the graph optimization.

<!-- 同时还可以用来做定位

首先做Visual Odometry and Submap Reconstruction, 来获取局部的位置估计, 并从images 估计出场景的3D geometric information. 当车辆经过infracture时, 车辆端获取到Infrastructure的静态场景点云, 并通过elastic registration的方式将visual submap与infrastructure point cloud 对齐. 然后, 将求解出的相对位置作为global constraint, 使用factor graph based method将视觉重建结果与infrastructure的定位结果进行融合, 在map merge之后得到一个global consistent map. -->



Submap Reconstruction
Associating the 2D image to a 3D point cloud is challenging. In this work, based on the motion prior of the vehicle camera, we reconstruct the road environment with keyframe-based sparse visual odometry, to obtain a 3D submap with homogeneous geometric information as the infrastructure point cloud for the 2D-3D association. We first extract FAST features from each image and track them between every two frames via KLT. Then, the frame-to-frame relative pose is estimated and the initial 3D coordinate of the feature points is recovered by triangulation. Due to the scale ambiguity of visual reconstruction, the imu measurement is also used to recover the initial scale of the submap. The 3D feature points(named landmarks) and their corresponding camera pose as well as the imu measurement are accumulated into a sliding window, and then the submap is optimized through then visual-inertial bundle adjustment. The structure of the submap can be described by the combination of three sets 
$$M = {\mathcal{T}, \mathcal{L}, \mathcal{O}}$$
where 
$$\mathcal{T}={T_i|T_i \in SE(3), i=1,...,n}$$
is the poses of each keyframes,
$$\mathcal{L}={L_i|L_i \in R^3, i=1,...,m}$$
is the coordinate of the estimate landmarks
and 
$$\mathcal{O}={<T_i, L_j, u, v>|T_i \in \mathcal{T}, L_j \in \mathcal{L}, u,v \in \mathbb{R}}$$ 
is the association between the poses and the landmarks, $<T_i, L_j>$ denotes the landmark $j$ is visible at camera pose $T_i$, and the observed pixel coordinate is $(u,v)$. The reconstruction is continuous during the vehicle driving, therefore the last pose $T_n$ can be an initial estimation of the current vehicle location, and the reconstructed points $L$ are kept for the subsequent submap registration and map optimization. 


<!-- Notice that the pose in $T$ is accumulated without global constraint, the error of each pose is also accumulated  -->

we can not obtain a global-consistant map by naively 

by accumulating the frame-to-frame relative pose
simultaneously


<!-- The reconstruction -->
<!-- 这个同时用来估计车辆的相对位置 -->
 <!-- use a local-matching-based approach to associate the submap and the infrastructure point cloud to capitalize on the initial localization informationfrom from vehicle, -->
<!-- 
 Thus the feature association can be searched in the local region of the visual submap and the point cloud, which is much more computing-efficient than the global matching methods. -->

There are two main challenges to align the point cloud with the submap. First, the submap are integrated by accumulate the visual odometry with the estimated 3D landmarks from 2D images, which means that the error of the pose, scale and feature depth are also accumulated into the submap and causing the skew and scale-inconsistency of the map.  Therefore, the conventional rigid-based point cloud registration algorithm is hard to be applied. An intuitive case is that, after tigid registration, part region of the submap could match, while displacement exists in the remaining part. Second, the visual submap is sparse and hard to extract effective geometric feature. By utilizing the non-rigid assumption, we proposed a elastic registration approach which jointly associate the non-rigid map segments with the infracture cloud and optimize the landmarks coordinate to tackle these problems, as described below.


static scene extraction


The point clouds obtained from the infrastructure LiDAR may be highly dynamic, especially for the scenario with heavy traffic flow. These dynamic points, such as vehicles and pedestrians, will cause the inconsistency between the visual submap and infrastructure point clouds. A straightforward idea to eliminate the influence is to classify the point cloud with semantic segmentation and then remove the points belonging to the possibly dynamic semantic category. However, inspired by \ref{removert} we observe that, based on the prior that the infrastructure LiDAR is mounted on a fixed position with a known pose, the dynamic points can actually be filtered out by the historical spatial occupancy information. Therefore, We divide the space that can be observed by the infrastructure into multiple voxels and then accumulate the point clouds during a period of time. The voxels are then voted if they are occupied by any points, and finally, those voxels with low occupancy can be classified as dynamic regions. The raw points that belong to those static voxels are subsampled and then transmitted to the vehicle. The details are shown in Algorithm \ref{}. And for the dynamic points in vehicle-side images, we remove the estimated 3D points with large reprojection errors before integrating them into the visual submap, since such points usually have an inconsistent motion with the static environment. \ref{vins}. The benefit of this design is that the succeeding submap registration procedure can obtain a time-invariant reference point cloud, and it improves the registration robustness. Besides, by doing this, there is also no need for the infrastructure to transmit the point cloud to the vehicle in real-time, since the point cloud is a static measurement of the environment and is identical during a period of time, and in the scenario where an infrastructure serves multiple vehicles, the same point cloud can be broadcasted to them, which greatly improves the scalability of the system.



time invariant/agnostic; 可扩展性

retains 那些非移动的动态障碍物 which may increase 特征数量并且b

对于车辆端的图像， 我们将visual odometry过程中重投影误差大的点从submap中剔除掉，因为这些点通常与静态环境有不一致的运动趋势 \ref{}

Initial Alignment 
When the vehicle drives near the infrastructure, we query the infrastructure localization in the pose set $T$ of the vehicle, extract the neighboring pose set $T_sub$ that is close to the infrastructure, and then transform the associate landmarks to the global frame to construct the visual submap $M$. Thanks to our factor-graph-based design, the initial localization $T_i \in \mathcal{T}$ can be firstly estimated by fusing the visual odometry with historical position constraints of infrastructure. For the special case where the vehicle hasn't been associated with any infrastructures, we can also estimate the initial pose by fusing the visual odometry with the GPS measurement or the TOA-range measurement from \textred{the infrastructure-vehicle communication device}. 

Due to the sparsity of the visual submap, the registration is first done on a coarser granularity with the aid of ground-plane semantic constraints. We estimate the ground-plane pose $T_{sub}$ from the submap and $T_{ipc}$ from infrastructure point clouds respectively. Notice the $x$, $y$ and $yaw$ part of the poses is zero since these axes are not observable from a plane estimation. Then we can estimate the pose that has eliminated relative roll, pitch, and z offset between the submap and infrastructure points

$$
T'_i = T_i \Delta T^{r,p,z} = T_i (T^{r,p,z}_{sub})^{-1} T^{r,p,z}_{icp}, ~T_i \in \mathcal{T}
$$
Taking $T'_i$ as an initial solution, we utilize the sparse NDT approach to estimate a rough relative alignment between the submap and infrastructure point cloud. For the sake of simplicity, we still use $T_i \in \mathcal{T}$ as the initial pose of the visual submap with respect to the infrastructure point cloud. Therefore, in the succeeding procedure, the feature association can be searched in the local region of the point cloud. It is more robust than the global matching methods since the visual submap is usually sparse and it is hard to extract distinctive geometry features. Meanwhile, it also improves computing efficiency because brute-force global matching is avoided.

elastic registration

Instead of treating the visual submap as a whole rigid point cloud, we regard it as a sequence of multiple 3D keyframe segments that may have displacement with each other. Each segment has an adjustable pose $\mathbf{T_i} \in \mathcal{T}$ and a corresponding 3D point set $\mathcal{F_i}$, representing the keyframe pose from initial alignment and the triangulated keyframe feature points attached with it. To associate the submap with the infrastructure point cloud, for each point in $\mathcal{F}_i$, we first search its neighboring point set from the infrastructure point cloud, if these points form a plane， then we estimate the parameter $(mathbf{n},mathbf{q})$, where $mathbf{n}$ is the normal vector of the plane and $mathbf{n}$ is an arbitrary fixed point on the plane. Then we can register the point clouds by minimizing the overall multi-segment point-to-plane distances

$$
r_d(\mathcal{T}, \mathcal{F}) =\sum_{ T_i \in \mathcal{T}}{ \sum_{p_j \in \mathcal{F}_i}{| \mathbf{n_{i,j}^T} \cdot ( \mathbf{T_i} \mathbf{p_j} - \mathbf{q_{i,j})} ) | } }  
$$
where $p_j$ is the 3D feature coordinate on camera frame that is observed at pose $T_i$, and $(\mathbf{n_{i,j},\mathbf{q_{i,j})$ neighboring plane parameter of $p_j$ from the infracture point clouds.

Meanwhile，$\mathcal{T}$ and $\mathcal{F_T}$ are also constrained by the visual reconstruction, which means that the projected coordinate of a landmark $p_j$ on each image(those who can observe $p_j$) with pose $T_i$, should be consistent with the pixel coordinate $u$ of the feature points that are directly extracted from the images, therefore we can also construct a visual measurement residual
$$
r_{vis}(\mathcal{T}, \mathcal{F}) = \sum_{ T_m\in \mathcal{T}}{ \sum_{ T_i \in \mathcal{T}} { \sum_{p_j \in \mathcal{F}_i} { \sigma_m^{ij} \lVert \pi(\mathbf{K}, \mathbf{T_m^{-1}} \mathbf{T_i} \mathbf{p_j}) - u_m^{ij} \lVert } } }
$$
where \sigma_m^{ij} \in \{0,1\} is a binary value denotes whether landmark $p_j$ can be observed at camera pose $T_m$, which is known from the feature matching during visual odometry; $\pi(\mathbf{K}, X}$ is a function that projects the 3D point $X$ to image pixel coordinate with \mathbf{K} as the camera intrinsic parameter.

Notice that, since a landmark may be covisiable in multiple images, it may has several corresponding $p_j$ \textcolor{red}{form ()()}, therefore
$$
\mathcal{L}\subset \{\mathcal{X}=\mathbf{T_i} \mathbf{p_j}|\mathbf{T_i} \in \mathcal{T}, \mathbf{p_j} \in \mathcal{F}_i \}
$$
Thus, rather than optimize all the 3D feature points from each frame, we only need to maintain a shared landmark set $\mathcal{L}$, so the residual can be simplified as

$$
r
\\
= r_{d}(\mathcal{T}, \mathcal{F}) + r_{vis}(\mathcal{T}, \mathcal{F})
\\
=r_{d}(\mathcal{T}, \mathcal{L}) + r_{vis}(\mathcal{T}, \mathcal{L})
\\
= \sum_{x_i \in \mathcal{L}}{| \mathbf{n_{i}^T} \cdot ( x_i - \mathbf{q_{i})} ) | }
+
 \sum_{ T_i \in \mathcal{T}} { \sum_{x_j \in \mathcal{L}} { \sigma_i^j \lVert \pi(\mathbf{K}, \mathbf{T_i^{-1}} x_j) - u_i^j \lVert } } 
$$

Then we can estimate the poses of each segment in the infrastructure global frame as well as the refined landmark coordinates by minimizing $r$
$$
\mathcal{T}_{reg}^*, \mathcal{L}_{reg}^* = \arg \min_{\mathcal{T},\mathcal{L}} r 
$$

The poses in $\mathcal{T_{reg}}^*$ are kept for the later map optimization as global constraints and the landmarks as well as are registered merged to the final map.


online map optimization

factor graph construction
To fuse the result from visual reconstruction and submap registration in real-time, we utilized a factor graph-based approach, as shown in Fig. {}. Each key-frame poses $T_i \in SE3$ to be solved are the node of the graph. The relative poses estimated by the visual odometry between adjacent frames are used as edge factors to connect each node from the graph. Meanwhile, the available GPS position $T_{gps} \in R^3$ that roughly estimated the vehicle position is also added to this factor graph as a constraint. Due to the large GPS positioning error, the covariance of this factor is set to a large value to against affecting the robustness of the initial positioning. 

After the submap registration, we use the optimized keyframe pose $\mathcal{T_{reg}}^*$ as prior factors and then add each factor to its corresponding node in the graph. Notice that, because the vehicle can not determine its world frame localization by visual odometry, we transform all of the visual odometry to the world frame after the first submap registration, then add a prior factor constraint at the first node and assign a large covariance to the translation and yaw angle part. In the subsequent optimization process,  the first pose is adjusted continuously to ensure the start point of the map is aligned with the world frame. 


map optimization
Once a certain amount of visual odometry or a submap registration constraint has been added to the factor graph, we periodically optimize the graph so that we find the Maximum A Posteriori (MAP) estimate for the keyframe poses. Then the poses from the graph node are extracted to transform the 3D landmarks of the corresponding keyframes to the world frame and fused as the final map. Moreover, the last graph node is the current estimation of vehicle localization, and also serves as the initial pose if the vehicle is driving through the next infrastructure. 

To maintain a global-consistent map and avoid map size increment during long-term mapping, the redundant feature points in the map should be eliminated. However, because the feature measurement varies when the camera observes the same position from different perspectives, the straight-forward voxelization downsample method may lead to the loss of multi-perspective feature information, which will cause tracking lost during re-localization. Therefore we design a map merging method based on local feature consistency, as shown in Algorithm~\ref{}. \textcolor{red}{Add brief description of the algorithm}.




The factor graph size increase continusly during the mapping process. To avoid repeated construction of optimization problems and to improve computational efficiency, we use Incremental Smoothing and Mapping (ISAM) to optimize pose. \textcolor{red}{Add brief description to the ISAM process}


在因子图在车辆定位建图的过程中是在持续增长的, to avoid 重复构建优化问题, 并且提高计算效率, we使用increamental Smoothing and Mapping的方式对pose进行优化.  在优化收敛之后, 我们
(可以通过优化residual的异常值发现outlier)


最终graph中节点的位姿被extract出来, 并且用这个pose将key frmae对应的3D landmark转换到世界坐标系, 构成最终的地图
M =

为了保持地图的一致性并降低长期建图过程中导致的地图尺寸增大的的问题, 应该对地图中冗余的特征点进行merge, 但是由于camera以不同实际视角观测同一个区域时, 特征测量值并不相同, 因此用straight-forward的voxelization方法或者random drop对地图下采样可能会导致地图特征的损失, 因此我们设计了基于局部特征一致性的地图合并方法, 如Algorithm所示
map merging 这里应该加


其中, 在submap registration之前, the estimated pose by the factor graph is used by section as initial pose



enpowers 大规模场景下使用低成本相机进行精确建图和定位的能力. 

beside, 错误的回环检测对SLAM系统是致命的, 可能会给地图的优化增加非常大的误差羡慕, 应用于驾驶场景是不可靠的



implementation

对于前端, 我们使用OpenVINS来获取到Visual Odometry, 以及三角化得到的特征点
使用ceres solver来优化elastic registration problem
我们使用gtsam framework来构建因子图优化框架


<!-- 最关键的一步是将 -->

<!-- 通过LM方法优化 -->

<!-- 查询每个点的邻域，然后估计平面，使用点到平面的距离来约束 -->


<!-- 第二个是稀疏的问题， 为了解决这个， 我们用 -->

<!-- since车辆是从上一个灯柱过来的,  -->

<!-- 由于submap是由frames累积起来的, submap在位置和尺度上与真实的3D点云相比都会有偏移, 用传统的刚体假设是无法求解精确的相对变换.因此我们提出 -->





<!-- 我们提出了基于factor fraph以及isam的方式来实时融合visual reconstruction以及submap registration的结果, 如图所示.   -->

<!-- 在submap registration之后, 我们extract优化之后的keyframe poses, 将其通过infrastructure的位置转换到world frame：然后将其作为prior factor 加入到graph中. -->

<!-- 
 我们在第一次与infrastructure配准之后, 将初始的odometry变换到infrastructure所对应的world frame, 并且我们在第一帧位姿上添加prior factor constraint, 并assign a large cov to the translation and yaw angle part, thus在后续的优化过程中, 通过不断的调整first pose to ensure the map 起点 is aligned with the world frame. -->



1. 传统的SLAM
<!-- SLAM算法的原理. -->
<!-- 连续的相机帧, 跟踪设置关键点, 以三角算法定位其3D位置, 同时使用此信息来逼近推测相机自己的姿态 -->
<!-- SLAM累计误差的原因, However, due to 1. 由于视角持续变化导致的特征点变化, frames之间feature 的matching和tracking会存在误差 2. 由于SLAM是基于静态场景的假设来求解相对位姿, 在图像中非静态区域的特征点会与静态区域有不一致的motion, 这样会对estimated的relative pose精度产生影响 3. 在实际部署中, 传感器内外参标定的误差会对特征点的测量精度产生影响, 特别是对于low-cost sensor like cameras. 因此这些误差在frame-by-frame的位姿估计中会被累计下来, 最终对long-term的localization产生不可避免的影响. -->

<!-- because when the vehicle is planning a path, whether it is manual planning or using the navigation algorithm, it will basically not drive through the similar path repeatedly in a short period of time. the way.
Besides, due to limited FOV of camera, 即使车辆经过同一个道路, loop detection也可能会fail due to slight change of 视角 of chaging lanes. -->
<!-- 
大部分SLAM方法使用loop closure detection\cite{LCD}来减轻累计误差的影响, 通过检测相机是否visit过同一个place, 构成loop constraint, 然后用pose graph optimization里eliminate累计误差. 但是这种方式对车辆运行的轨迹有严格要求, 只有当车辆当前位置与历史位置有large overlap, 才有可能detec 到loop closure, 

但是在实际的驾驶场景中, 这样的loop constraint是难以实现的, 因为从直观上来讲, 车辆在规划路径的时候, 无论是人工规划还是使用navigation algorithm都基本不会在短时间内驾驶反复经过相同的道路. 

Besides, 由于limited FOV of camera, 即使车辆经过同一个道路, loop detection也可能会fail due to slight change of 视角 of chaging lanes. -->


<!-- 我们在车辆上部署了camera set up a road test to verify the performance of conventional slam algorithms -->
<!-- 即使形成回环也是非常稀疏的, 误差的累计难以通过回环完全消除; -->
<!-- 图示是一个常见的乘用车驾驶轨迹, Z轴方向的高度偏差, 并且解释, 由于图像的视角没有重叠, 回环约束无法形成, 车辆在整个过程中都处于dead rocking过程, 误差在不断地累计 -->
<!-- 这样的驾驶轨迹很常见, -->


<!-- 在车辆行驶的过程中, 我们使用ORB-SLAM3来进行车辆的位姿估计, 并将重建的3D特征点进行融合后得到稀疏的特征点地图, as shown in Figure. \ref{fig:motivation_study}, 其中地图点的颜色代表驾驶时间. It can be seen that, 由于缺少全局约束, there is an obvious accumulated drift in the estimated pose and map by ORBSLAM3 compared with the ground truth trajectory. -->

<!-- 此外地图的尺度也出现了偏移, 可以看出重建的地图的width大于ground truth trajectory -->

<!-- 车辆两次经过时的orientation是完全相反的, 此时camera images完全没有重叠的measurement 来检测loop. -->

<!-- This case demonstrates that the conventionalSLAM难以得到全局一致性的定位 在驾驶场景下, 我们也调研了GPS定位, 以及SLAM融合GPS的精度, 如图所示, (绝对误差评估, 累计误差评估) -->


<!-- only needs 传场景的静态特征 -->

<!-- therefore the map-centralized approaches such as ORB-SLAM \cite{campos2021orb} may not work as excepted, -->


<!-- Currently, one of the most widely used low-cost sensors on mainstream vehicles are the cameras,  while camera-based SLAM -->

<!-- the widely utilized loop closure detection -->

<!-- 高质量的global constraint -->


The basic idea of visual SLAM front-end is to track the associated feature points from consecutive frames and use the triangulation method to recover the 3D positions of the points, and then estimate the ego-poses of the camera through multi-view geometry. By accumulating the estimated poses and 3D points, the localization of the camera relative to the local frame can be obtained, as well as the environment map. However, because (1) the feature descriptors of the landmarks consistently change with the camera perspective, the feature matching and tracking process are not stable; (2) since SLAM is based on the static environment assumption, the features in the non-static area will have inconsistent motion with the static area, which also affects the accuracy of relative pose estimation; and (3) in actual deployment, the intrinsic and extrinsic sensors calibration error decrease the measurement accuracy of feature points, especially for a low-cost sensor like cameras. These errors will be inevitability accumulated during frame-by-frame pose estimation, thus making the accuracy of localization get worse during long-term driving. Most SLAM methods utilize loop closure detection\cite{LCD} to tackle this problem, which detects whether the camera has visited the same place before by image matching and constructs loop constraint, and then eliminates the error by pose graph optimization. However, this method has strict requirements on the vehicle trajectory, only when the current and the historical location of the vehicle have a large overlap can the loop closure be detected. In actual driving scenarios, such a loop constraint is hard to achieve, because the vehicle barely drives through a similar path repeatedly during a short period. Moreover, due to the limited FOV of the camera, the loop detection may still fail because of a slight change of perspective or changing lanes, even though the vehicle drives back to the same road.

We set up a real-world road test to evaluate the performance of conventional slam algorithms. A RealSense D455 (90-degree FOV, with an imu) camera is mounted on the front of an autonomous vehicle, and a high-precision RTK-GNSS system is used to generate ground truth trajectory. We use ORB-SLAM3 \cite{} to estimate the camera pose while the vehicle is driving along the road, and then fuse the reconstructed 3D landmarks to obtain a sparse feature point map, as shown in Figure. \ref{fig:motivation_study}, where the color of the map points represents driving time. It can be seen that, due to lacking global constraint, there is an obvious accumulated drift in the estimated pose and map by ORBSLAM3 compared with the ground truth trajectory. At point A, the reconstructed ground plane of the two roads ought to overlap with each other since the vehicle is driving on a 2D road, however, there is an obvious Z-axis map drift at around 3m; then the error accumulates continuously, and at Point B, the drift has reached around 12m at Z-axis and 8m at Y-axis. Besides, the scale of the map has also shifted, which can be seen from the left part of the map, the width of the reconstructed map is slightly larger than the ground truth trajectory. The loop closure detection fails either at point A or B because the current and the historical orientation mismatches, especially at B, the two vehicle orientations while passing the road are completely opposite, therefore there are not enough overlapped features to detect loop closure.

On the other hand, it can be seen that the ground truth trajectory aligns well with the point clouds from the infrastructure, whose location has been pre-calibrated during the installation. The LiDAR mounted on the infrastructure can obtain an accurate 3D measurement of the surrounding environment. Taking the Livox Horizon as an example, its ranging accuracy is about 2cm within a range of 100m. If this measurement can be matched with the 3D map reconstructed by the vehicle, the precise location constraints can be obtained and utilized to optimize the SLAM map. Since the observation of infrastructure is real-time, the map can be updated with the latest environmental information every time the vehicle passes the infrastructure to maintain an up-to-date and global-constraint map. And the vehicle can also perform re-localization on this map to estimate precise relative pose in the infrastructure frame and thus improve the perception range of the vehicle.   

The red points are the point clouds from the infrastructure; the purple-to-yellow colored points are the 3D map reconstructed by conventional SLAM algorithm, where the color denotes increasing driving time; the grey dotted line is the ground truth trajectory of the vehicle. 



1. Benifits of infrastructure-assisted mapping

相反, 在图中可以看出, 车辆的Ground Truth与Point cloud from infrastructure, whose location has been pre-calibrated during installation. 

RSU上的LiDAR可以获取到周围环境的精确measurement. 以Livox Horizon为例, 其在100米范围内的测距精度可以达到2cm. 如果可以将这一测量值与车辆观测到的环境信息进行匹配, 那么就可以得到精确的位置约束, 并用来优化SLAM map. 并且由于RSU的观测量是实时的, 每次车辆经过RSU的时候都可以以最新的环境信息修正地图, 保证地图的及时性


当有这个地图之后, 车辆在其上进行re-localization, 即可获取到对齐infrastructure的精确定位, 

车辆此时就可以融合RSU和车辆共同的观测量, 补充车辆视角的盲区. 但是, 这样的数据融合依赖于车辆和RSU之间精确的相对定位


同时, 由于infrastructure提供的全局约束是作用在整个SLAM过程中的所有历史keyframs上的, 因此在地图优化之后, 那些infrastructure覆盖不到的区域的地图也可以得到修正.


除此之外, 如果RSU部署的足够密集, with RSU上部署的LiDAR seosors, 可以提供一个实时的三维高精度地图, 


Thanks to our design, 我们可以给submap配准一个比较好的初始值, 所以可以用局部性的方式来配准

introduction

1. 地图很重要
2. 但是现有的高精地图都是图商提供的, 构建地图需要高成本sensor, it is also required by要在这些地图上定位的用户车辆.
3. 为了保证地图的实时性,图像需要periodcally capturing mapping data, and the automous vehicle thus need to frequently update large amount of map data. 此外, 有大量的园区,停车场,私家路等小型场景是图商无法覆盖到的 service is unavailable.
4. Therefore, 所以车辆自己维护一个up to date的high quality地图是十分必要的, but it's challenging for 消费级的车辆用低成本的传感器来构建地图
5. 列举一些SLAM方法, 累计误差问题, 融合GPS问题
6. on the other hand, the develop of smart infrastructure enables 高质量的三维数据获取 from road side, by exploring 这些信息, 可以获取到RSU附近的精确三维信息
7. 但是大部分工作还是集中在road-side的交通流监控这类的应用上, 而不是提高无人驾驶系统的性能
8. In this paper, we present VILM, a Vehicle-Infrastructure cooperative Localization and Mapping Framework

The high-definition 3D map is essential to enable highly automated driving. However, existing mapping frameworks \cite{ilci2020high} generally rely on high-cost sensors, e.g., LiDARs and GNSS/inertial navigation systems, making it difficult for mass commercial deployment and real-time updates. The map maintainers usually employ fleets of vehicles equipped with surveying-level sensors, and these vehicles scan the road environment periodically to keep the map up-to-date, thus the vehicles need to update massive map data frequently. Besides, there are a large number of scenarios, such as industrial parks, private roads, and parking lots, that can not be covered by such mapping services currently. Therefore, it is necessary for the autonomous vehicle to maintain a high-quality and up-to-date map itself with low-cost sensors like cameras. 

The camera-based Simultaneous Localization and Mapping (SLAM) \cite{campos2021orb, labbe2019rtab, qin2018vins} provides a feasible solution to this issue, but this approach suffers from the inevitable accumulated error after long-term mapping, due to the degenerated sensor measurement and lacking global constraint. Most SLAM algorithms utilize loop closure detection\cite{arshad2021role} to eliminate this error, however, in a large-scale driving scenario, the vehicles merely drive through the same roads from a similar perspective repeatedly during a short period, and that significantly increases the failure probability of loop detection. Thus, the key to improving the mapping accuracy is to provide high-precision global constraints to eliminate the cumulative error. The increasingly deployed intelligent roadside infrastructures, such as lampposts equipped with LiDARs, enable the acquisition of high-quality 3D measurement of the road environment, by associating the 3D data with measurement from the vehicle, one can estimate accurate relative transformation between the infrastructure and the vehicle \cite{he2021vi, hossain2019cooperative}. As the localization of the infrastructure can be pre-calibrated during installation, the relative pose information can be transformed to a world frame and serve as a high-precision global constraint to mapping accuracy.

In this paper, we propose VILM, a Vehicle-Infrastructure collaborative 3D Localization and Mapping framework, to enable high-quality and low-cost vehicle-side visual mapping and localization, by fusing the 3D measurement from the intelligent road-side infrastructure. We designed an elastic submap-to-frame registration algorithm, to associate the vehicle camera images to the infrastructure point clouds, and then present a factor graph-based map optimization method to fuse the vehicle-infrastructure matching information to reconstruct a global-consistent map. The mapping framework can be performed in real-time with a low-cost communication, which only needs to transmit a static scene point cloud from the infrastructure to the vehicle once.  We evaluate the proposed system on extensive datasets under various driving scenarios and prove the accuracy and robustness of the proposed framework.



<!-- Currently, one of the most widely used low-cost sensors on mainstream vehicles are the cameras, while camera-based SLAM -->

Specifically, the contributions of this paper can be summarized as follows:
1. An infrastructure-vehicle collaborative mapping system is proposed, which enables low-drift and global-consistent localization and mapping for autonomous vehicles with only low-cost cameras. 
2. An elastic 3D-to-2D registration algorithm is proposed to associate the camera images to the point cloud measurement from the infrastructure.
3. We implement and benchmark the proposed system on extensive self-collected datasets, and prove the mapping accuracy and robustness under various driving scenarios.

elastic 

我们首先用voxelized来做初始对齐



1. SLAM前端, VIO marginalization 之后, 最后的keypoints取出来
2. 静态场景过滤
3. 匹配 
   1. 使用语义信息获取初始解
   2. BA photoggeometric loss + point-on-plane constraint, multiple T
4. 构建factor graph, 画一个graph
5. 地图优化, 在factor graph optimization之后, 以新的位姿作为初始值, 检查一致性


As mentioned in Section 4, during the mapping process, VIML maintains the optimized pose with the factor graph， therefore we can obtain a continuous trajectory of the vehicle by extracting the poses from each corresponding node. Then we calculate the ATE and RTE between the groundtruth and the estimated trajectories to evaluate the mapping accuracy. We also calculate the relative translation error which measures the drift of the estimated trajectories. We compare our approach with mainstream visual SLAM methods, the OpenVINS is run in visual odometry mode without loop closing, the VINS-Fusion and ORB-SLAM3 are run with loop closing correction, and VINS-GPS is a method that fuses the visual odometry with the GPS measurements as the global constraint. It is worth mentioning that the OpenVINS, VINS-Fusion, and ORB-SLAM3 can only estimate poses in the local coordinate frame w.r.t the start point of the map， while the groundtruth is GNSS pose in the UTM frame. Therefore, we need to first match the estimation with the groundtruth trajectory and transform it to the global frame before evaluation. In practice, poses in the UTM frame are also required in the vehicle localization task and there is no groundtruth reference provided, and it remains to be a challenging problem how to align the estimation from SLAM method like ORB-SLAM3 to the global frame in such a scenario. Our approach can directly generate maps and pose estimation under the global frame by associating with the infrastructure, which proves from another aspect that our method is more consistent with downstream tasks.

As shown in Table. \ref{}, the OpenVINS and VINS-VIO perform worst because they are running in odometry only mode, and the localization error are accumulate without correction. And the performance of loop-closing-based methods such as VINS-Loop and ORB-SLAM3 are close to the visual-odometry methods because the loop closing constraints are sparse in the testing scenario and are not enough to eliminate the accumulative errors. \textcolor{red}{Adding specific description of the scenario.} The VINS-GPS is more globally consistent, but due to the multi-path effect in the urban environment, the variance of the GPS measurements is large. In this case, the confidence of the GPS poses in the fusion algorithm is reduced. Besides, there are also GPS-denied environments overpasses and long tunnels. Therefore, the localization performance still relies on visual odometry in most cases and also suffers from accumulative error. VIML outperforms the other methods, by periodically registering with the infrastructure point clouds, it can eliminate the cumulative error in time and align the constructed map to the global consistent coordinate system. Besides, VIML can also work in GPS-denied environments where the infrastructures are deployed and ensure that the vehicle has a reliable and high-precision localization in such scenarios.

VIML outperforms其他方法， 通过周期性的与infrastructure点云进行配准，消除了累计误差，并且将地图对齐到了UTM一致的全局坐标系下。Besides， VIML同样可以在部署了infrastructure的GPS-denied环境下工作，可以在这种情况下让车辆也有可信赖的高精度定位

ORB-SLAM3和VINS-loop的性能接近，因为车辆轨迹中loop closing的case太稀疏. 

VINS-GPS更具有全局一致性，但是由于GPS测量值在城市中由于多路径效应，方差较大，在这种情况下只能将融合算法中的GPS置信度降低，因此大部分情况下还是依赖visual odometry. Besides, 在立交桥和长隧道这样的GPS-denied环境，还是只能依靠visual odometry进行定位，累计误差依然存在.


% 1.  As mentioned in Sec4, 在，mapping过程中， VIML通过因子图来维护车辆经过infrastructure优化之后的pose， 因此将因子图的每个节点对应的pose取出来之后， 可以得到连续的轨迹, 我们通过计算其与groundtruth之间的ATE以及RTE来评估mapping accuracy

% 1. 结果如表所示， 我们与主流方法进行了对比。 ORBSLAM3自带回环， OpenVINS是VO模式，VINS-FUSION是VO加回环检测，VINS-GPS是VO融合了GPS

% LIO-SAM是LiDAR建图的结果，需要一个高成本的32线激光雷达， which还难以在大规模的驾驶任务中使用， 而我们的方法已经接近LiDAR建图的效果， 在全局一致性的结果上还超过了LiDAR

% 注意ORB-SLAM3， VINS以及OpenVINS只能输出相对于建图起点的局部坐标系下的位姿， 而我们的groundtruth是在UTM坐标系下的，因此我们使用【evo align】将轨迹匹配到grountruth所在的坐标系，然后再进行评估。

% 在真实的车辆定位任务中通常需要的也是UTM系下的坐标，而这时是没法获取到gt的， 怎么将ORMSLAM方法的定位转换到到全局坐标系下还是一个challenge problem。 我们的方法可以直接产生这一系下的地图和位姿，这从另一个方面证明，我们的方法这与下游任务更有consistency。

% 从中可以看出，纯VO模式的open-vins与VINS的效果最差， 因为误差持续累积；ORB-SLAM3和VINS-LOOP虽然开了回环检测， 但是由于数据集是模拟真实车辆的驾驶轨迹， 轨迹中产生的回环十分稀疏，通常只有两种情况，第一种是整个轨迹是长路like 立交桥，误差持续累积；第二种是经过很久才回一次环，回环约束不足以消除所有的误差

% （这里再加上GPS本身的error）VINS-GPS更有全局一致性， 但是由于GPS测量值在城市中，一部分由于多径效应有很大的跳变， 这种情况下只能将融合算法中GPS的置信度降低，大部分还是依赖VO，另外像立交桥和长隧道这种只能通过VO累积，误差仍然很大

% VIML outperforms其他方法， 通过周期性的与infrastructure点云进行配准，消除了累计误差，并且将地图对齐到了UTM一致的全局坐标系下。Besides， VIML同样可以在部署了infrastructure的GPS-denied环境下工作，可以在这种情况下让车辆也有可信赖的高精度定位


为了使用下游任务验证构建地图的质量，我们优化之后的地图上进行了重定位实验。我们从因子图中extract出每个节点对应的位姿T，并从特征点集合中找到T对应的特征点。为了节省存储空间，我们不保存每一个keyframe的原始图像，而是



$$
r_{vis}(\mathcal{T}, \mathcal{F}_i) = \sum_{ T_m\in \mathcal{T}}{ \sum_{ T_i \in \mathcal{T}} { \sum_{p_j \in \mathcal{F}_i} { \sigma_m^{ij} \lVert \pi(\mathbf{K}, \mathbf{T_m^{-1}} \mathbf{T_i} \mathbf{p_j}) - u_m^{ij} \lVert } } }

\\
r_d(\mathcal{T}, \mathcal{F}_i) =\sum_{ T_i \in \mathcal{T}}{ \sum_{p_j \in \mathcal{F}_i}{| \mathbf{n_{i,j}^T} \cdot ( \mathbf{T_i} \mathbf{p_j} - \mathbf{q_{i,j})} ) | } } 
\\
\mathcal{L}\subset \{\mathcal{X}=\mathbf{T_i} \mathbf{p_j}|\mathbf{T_i} \in \mathcal{T}, \mathbf{p_j} \in \mathcal{F}_i \}
\\

r(\mathcal{T}, \mathcal{L}) 
\\
= r_{d}(\mathcal{T}, \mathcal{F}) + r_{vis}(\mathcal{T}, \mathcal{F})
\\
=r_{d}(\mathcal{T}, \mathcal{L}) + r_{vis}(\mathcal{T}, \mathcal{L})
\\
= \sum_{x_i \in \mathcal{L}}{| \mathbf{n_{i}^T} \cdot ( x_i - \mathbf{q_{i})} ) | }
+
 \sum_{ T_i \in \mathcal{T}} { \sum_{x_j \in \mathcal{L}} { \sigma_i^j \lVert \pi(\mathbf{K}, \mathbf{T_i^{-1}} x_j) - u_i^j \lVert } } 
\\
\mathcal{T}^*, \mathcal{L}^* = \arg \min_{\mathcal{T},\mathcal{L}} r
\\
T'_i = T_i \Delta T^{r,p,z} = T_i T^{r,p,z}_{sub}^-1 T^{r,p,z}_{icp}, ~T_i \in \mathcal{T}

$$