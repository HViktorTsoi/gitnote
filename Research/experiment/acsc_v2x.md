# carla v2x数据集生成
1. LiDAR的intensity, 用RGB转换成grey之后索引, 或者融合semantic标注, 对车道线路面等赋予一些特殊的intensity值
2. 对label 进行反射率纹理映射
3. rsu要放在什么位置, 这个最好能够自动实现部署, 或者至少可以人手工指定一些位置
4. 怎么实现车辆沿着一定的waypoint经过指定的rsu. 参考https://github.com/carla-simulator/carla/issues/3890, 以及carla的PythonAPI/agent实现
5. 怎么获取相机的内参, 怎么获取vehicle-cam到rsu-lidar之间的tf

# opensfm
尝试在Py27安装

# 一个观察到的现象: 以灯杆作为reference, 由于重建的点质量较差, 配准效果差; 而如果以重建作为reference, 配准的效果就更好