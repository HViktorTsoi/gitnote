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

# 考虑用地面上的竖直杆子作为约束
