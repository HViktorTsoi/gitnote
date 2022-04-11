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
1. 跑VIO， 将现有的需要地图优化的Keyframe Odometry，以及对应的image， /vins_estimator/keyframe_point
2. 注意/vins_estimator/keyframe_point里的point3d是VIO世界坐标系，如果经过gtsamleKF的位姿，那么point3d对应的点坐标也应该优化，保持一致
3. 保存VINS格式的pose graph， 其中VIO和PG都是优化得到的POSE， LOOP INFO全部设置为0(LOOP INFO存的是之前回环优化之后的pose和原始VIO之间的相对值, 格式为x,y,z,qw,qx,qy,qz,yaw); loop index是当前帧所回环到的历史帧ID, 默认都设置成-1
4. 保存KF对应的keypoints以及descriptors, 注意分为两部分， 一个是VINS给的， 一个是LoopFusion自己重新检测的。 为了获得这些信息，需要过一遍KeyFrames的构造函数
5. 1~4的过程可以考虑将优化之后的Pose以及更新的keyfram_point, image作为伪VINS的输出给loop_fusion, 然后使用loop_fusion的保存功能将pose graph存下李
6. 在定位过程中， 首先载入保存的posegraph； 然后等待loop fusion将当前帧的与先验地图进行loop， 来做全局定位； 这里可能需要关掉optimization（如果只要最新的localization结果）



1. 传统的SLAM
<!-- SLAM算法的原理. -->
<!-- 连续的相机帧，跟踪设置关键点，以三角算法定位其3D位置，同时使用此信息来逼近推测相机自己的姿态 -->
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

<!-- only needs 传场景的静态特征 -->

<!-- therefore the map-centralized approaches such as ORB-SLAM \cite{campos2021orb} may not work as excepted, -->


<!-- Currently, one of the most widely used low-cost sensors on mainstream vehicles are the cameras,  while camera-based SLAM -->

the widely utilized loop closure detection

高质量的global constraint





Specifically, the contributions of this paper can be summarized as follows:
1. An infrastructure-vehicle collaborative mapping system is proposed, which enables low-drift and global-consistent localization and mapping for autonomous vehicles with only low-cost cameras. 
2. An elastic 3D-to-2D registration algorithm is proposed to associate the camera images to the point cloud measurement from the infrastructure.
3. We implement and benchmark the proposed system on extensive self-collected datasets, and prove the mapping accuracy and robustness under various driving scenarios.


