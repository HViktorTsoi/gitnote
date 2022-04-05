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