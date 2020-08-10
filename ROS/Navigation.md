1. gmapping建图需要的tf：sens
2. obscale layer: 避障层, 其中的没有inflation radius；
2. inflation_radius: 静态地图的膨胀层，用在global mao上, 其中的inflation_radius是地图膨胀计算的半径，在此半径内，用一个指数函数来计算碰撞的cost，即这个半径并不是robot要躲避的半径，而是躲避半径函数的最大值；
3. localization，需要稳定的里程计，之前使用lego loam作为里程计会经常抖动，结果导致路径规划器的规划出现高频抖动，控制效果非常差；换成差分底盘的轮式里程计之后，规划的轨迹和效果都非常稳定
4. odometry可能时时刻刻都有累计误差，因此这个时候就要加上时时刻刻运行的amcl，可以保证里程计大概准确的情况下提供精准的定位；
5. 为了将lego loam作为里程计使用，需要改这几个地方：
	1. launch里边，camera_init_to_map这个static tf要改成camera_init_to_odom，发布 camera_init 到 odom 的tf
	2. transformFusion里，map_2_camera_init_Trans.frame_id_要改成odom

