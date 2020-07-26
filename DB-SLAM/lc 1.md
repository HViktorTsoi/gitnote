## SLAM分类
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/07/26/1595757558715-1595757558729.png)
1. 静态SLAM(场景物体不会变化)
	1. 尺度地图：和真实世界尺度(尺寸)一样大小的地图；
		1. 基于bayes(滤波器)的构建
		2. 基于图优化的构建
	2. 拓扑地图：简化成图的地图，各个节点之间能够到达是用边来连接，适用大的场景，用于下一步的路径规划； 
	3. 混合地图：结合1,2，在topo地图的每一个节点上都有一个真实size的尺度地图
2. 动态SLAM(场景物体会发生变化)

## Graph-based SLAM
分为前端和后端，前端是构图，后端是优化过程
节点：机器人位姿
边：节点之间的空间约束关系

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/07/26/1595757931647-1595757931650.png)
想要实现图优化，就必须有类似回环的东西。比如图中从节点1走到节点6，不同的观测值构成了不同的节点，如果到节点6的时候，发现1和6有部分观测值是重合的，就可以求出1和6之间的位姿T2；而顺着链条一步一步求出来的1~6的位姿是T1,那么理想情况下T1的逆和T2之间积应该是I。但实际上两者之间肯定存在误差。由于一个图中存在多个环，那么就可以用优化减小这个误差了，优化的变量就是每个节点上的位姿，最终使所有图上的环都满足误差最小。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/07/26/1595758976384-1595758976387.png)
例如上图，从Xi到Xj有两条路径，下边是根据图上一个个观察节点累计得到的结果，上边是由重叠部分直接观测得到的，那么图优化的目标就是去优化一个误差向量e,使得由两种方法得到的Xj能够最好的重合。

## Filter based SLAM
基于滤波的方法，特点是只估计基于T时刻的位姿，但是累计误差很严重，基本步骤如下：
1. 状态预测
2. 测量预测
3. 进行测量
4. 数据关联
5. 状态更新&地图更新
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/07/26/1595760928034-1595760928035.png)