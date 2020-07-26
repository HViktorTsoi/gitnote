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
边：