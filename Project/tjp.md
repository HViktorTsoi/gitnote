# Note
## hdl localization
1. IMU LiDAR 外参标定

## LIO localization
跑Dan Hai的定位算法
1. 方案1: 
   1. 忽略odom to map的估计
   2. 直接把global initialization作为map optimization的起始odometry
   3. 在每次全局定位之后, 把全局transform作为gps factor, 加入因子图进行优化
2. 方案2:
   1. 维护odom to map;
   2. 每次全局定位只有, 把全局transform作为gps factor, 加入因子图进行优化. (验证是不是要把全局定位转换到odometry系再加到因子图中)

## 开发日志
1. lio localization中, 用滑动窗口win来保存待全局匹配的关键帧, 并且没有保存关键帧id, 如果在全局定位之后直接用win的size来作为因子图节点的ID, 那么这个ID
ID就会是错误的, 在超过滑动窗口大小之后, ID就一直是那个窗口最大长度了
2. 在全局定位之后, 求odom测量值时, 旧的odom2map的tf不一定与关键帧是对应的,这里需要再确定一下
3. 尝试了只使用初始的map to odom(后续不更新), 这和将odom的第一帧设置为map中的绝对位姿是一致的, 结果发现,轨迹确实不会有跳变了, 但是变成低频的凸起了, 这可能说明全局定位的观测量就是不准的
4. reply速度对性能也有影响, 实验发现mapping的速度总会落后odometry几帧 **这是因为我开发的时候关闭了keypose的滑动窗口,现在每次回环检测都要遍历所有的滑动窗口, 待修复**
5. 飞了的情况应该是mapping测量得到的相对距离比imu预积分得到的距离大了, 导致优化出来的ba比实际的大很多, 在后续的积分上,用一个特别大的ba去积分, 当然位姿就飞了
6. 优化之后odometry还是很平滑的， 我们看到的跳变是map to odom 造成的
7. 思考， 为什么用lio relocalization的原始形式， 得到的轨迹不是平滑的？
   1. 如果优化odom to map， 则这个硬性的定时优化当然会影响平滑性
   2. 如果只优化odom， 为什么会轨迹会慢慢远离map？ 第一个因子是0， 在计算的时候转换到map， 与第一个因子是odom2map的区别是什么？？？
8. 我们需要全局轨迹优化吗？ 全局匹配的效果很好， lio的累计误差也不大，是不是不需要优化太久之前的轨迹？？？