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

With the recent widespread application of LiDAR, there are a series of studies on the fusion of LiDAR point clouds and camera images， especially for the perception \cite{qian2021robust,vora2020pointpainting, ouyang2022saccadefork} and localization \cite{debeunne2020review} tasks of autonomous driving. 

R2LIVE \cite{lin2021r} proposed a tightly-coupled sensor fusion framework, which fuses measurement from LiDAR, inertial sensor, and visual camera to achieve robust and accurate state estimation. The motion state within the framework is estimated by error-state iterated Kalman-filter to guarantee real-time performance and then refined with factor graph optimization. 

LVI-SAM \cite{shan2021lvi} proposed a semi-tightly-coupled lidar-visual-inertial odometry via smoothing and mapping, which improves the accuracy of the visual odometry by extracting depth information for visual features from lidar measurements, and conversely utilized the visual odometry as initial estimation for LiDAR Mapping, to improve the robustness in the geometric degenerated environment. 

\cite{zhao2021super} employs an IMU-centric fusion pipeline to bridge the information from the camera and LiDAR, which combines the advantages of loosely coupled methods with tightly coupled methods and recovers motion in a coarse-to-fine manner.

 Most of the existing fusion approaches assume that the LiDAR and camera are mounted to a fixed base frame, and also strictly require sensor measurement to share the same FOV. Such methods are hard to be applied in vehicle-infrastructure fusion scenarios, where the relative position between the vehicle and the infrastructure constantly changes, and they do not share the similar FOV in most cases. Besides, the cost of equipping integrated LiDAR-camera sensors on vehicles is still high, which also limits the mess deployment of autonomous vehicles. In this paper, we propose a new paradigm for LiDAR-camera fusion, which only requires low-cost cameras mounted on vehicle, and can achieve high-precision localization and mapping by fusing the point cloud information from roadside infrastructures.

%=====================================================================

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
}

Background Filtering

Inspired By [], we propose a background filtering pipeline

对比实验, 把全局的轨迹误差和靠近RSU的轨迹误差放在一起对比

scalability, 因为计算是在车端, 并且所有车端接收到的数据都是一样的, RSU只需要隔一段时间记录自己周围的环境变化即可； 如果RSU部署的多, 车辆相当于得到了近似实时的高精地图

notice that the driving direction there是相反的, 在这里仅仅靠图像是无法形成回环约束的

use only Infrastructure


Submap Reconstruction
Associating the 2D image to 3D point cloud is challenging. In this work, 我们基于the motion prior of vehicle camera， 使用图像对道路环境进行Keyframe-based sparse reconstruction, 来获取3D submap with 同质的information with the infrastructure point cloud。由于visual reconstruction存在尺度不确定的问题， the imu measurement is also used to recover the initial scale of the submap. 我们首先对extract FAST feature points for each images and track the features between each two frames via KIT. Then, the frame-to-frame realative pose is estimated and the initial 3D coordinate of the featue points is recovered by triangulation. The 3D feature points(named landmarks) and its corresponding camera pose as well as the imu measurement are accumulate into a sliding window, and then the submap is optimized through visual-inertial bundle adjustment. The structure of the submap can be described by the combination three sets 
M = {T, D, O}
where T={T_i|T_i \in SE(3), i=1,...,k_T} is the poses of each key frames,
D={D_i|D_i \in R^3, i=1,...,k_D} is the coordinate of the estimate landmarks
and O={<T_j, D_k， u，v>|T_j \in T, D_k \in D} is the association between poses and the landmarks, <T_j, D_k> denotes the landmark $k$ is visiable at camera pose $T_j$, and the observed pixel coordinate is (u,v). 

注意这个reconstruction过程是持续进行的, 因此车辆可以同时估计自己相对于起始点的odometry
The reconstruction
这个同时用来估计车辆的相对位置

initial localization
为了充分利用初始定位信息from vehicle，并节省计算时间，我们设计了局部匹配的方法，来关联submap和infrastructure point cloud. Thanks to our factor-graph-based design， the rough realtive localization can be estimate by fusing visual odometry with historical position constraint of infrastructure. In particular, for the initialization case where the vehicle hasn't been associated to any infrastructure, we estimate the initial pose by fusing the visual odometry with the GPS measurement and the TOA-range measurement from \textred{the infrastructure-vehicle communication device}. 

Thus the feature association between visual submap and point cloud can be performed localy， and much more computing-efficient than then global matching based method.
since车辆是从上一个灯柱过来的， 


由于submap是由frames累积起来的， submap在位置和尺度上与真实的3D点云相比都会有偏移， 用传统的刚体假设是无法求解精确的相对变换。因此我们提出


online map optimization


enpowers 大规模场景下使用低成本相机进行精确建图和定位的能力。

beside， 错误的回环检测对SLAM系统是致命的，可能会给地图的优化增加非常大的误差羡慕， 应用于驾驶场景是不可靠的
