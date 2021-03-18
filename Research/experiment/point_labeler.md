# Point Labeler
## 安装

按照https://github.com/jbehley/point_labeler的文档安装。

注意最好单独用一个catckin的workspace以防止环境污染。

详细的操作文档在https://github.com/jbehley/point_labeler/wiki

## 数据组织形式和配置

1. 数据格式和KITTI odometry完全一致，数据目录下需要以下内容
   1. `velodyne/`: 所有的点云
   2. `poses.txt`: 每一帧点云对应的位姿，格式见KITTI odometry
   3. `calib.txt`: LiDAR到camera的外参，注意即使没有4，也需要有这个文件
   4. `image_2/`: （可选）点云对应的图像，用来在标注的时候作为参考

2. 配置文件`settings.cfg`在编译完成后的`bin/`目录下，和`labeler`二进制文件在同一个目录，其中
   1. `tile size`: 单位是米，将整个数据集划分成多个`tile size`*`tile size`的瓦片，标注过程中当前屏幕中显示的点云就是一个瓦片中叠加的所有点云；
   2. `max scans`: 一次性载入的所有点云，根据自己的显存调整；如果你的显存过小导致这个值过小，那么一个tile就无法充满点，有一部分点云无法被标注，这种情况下或者找一个显存足够的机器，或者减小`tile_size`(但这样会增加标注工作量)；
   3. `min range`, `max range`: 最大最小范围， KITTI大概设置的最大范围是80米；


## 数据预处理

1. poses.txt中的位姿需要尽量准确，否则叠加出的点云方差过大，并且如果叠加帧数较多时，不同物体之间会有重叠，
2. 目前看，m39给出的位姿并不够精确，考虑使用LIO-SAM的位姿，或者m39+ICP refine之后的位姿
3. 点云的运动畸变影响较大，尝试用消除畸变之后的点云做标注，消除畸变的点云和原始点云采用同样的顺序存储，只不过xyz数值被修正；
![20210318163018](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20210318163018.png)

4. 将点云对应的图像一起保存，标注时作为参考
5. 关于最大距离最小距离的选取，根据KITTI odometry的统计数据
   1. 每个序列的最大距离为：80.68931；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325；81.352325
   2. 即KITTI odometry，3D segmentation所有点云最远距离仅有约80m ![20210318152851](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20210318152851.png)
   3. 所以目前可以把最大距离设置为100m，100~200以上的点云暂时丢弃，并且由于运动畸变这部分点云会对叠加结果产生影响

# 标注类别

1. 目前的版本，就使用point_labeler给定的类别，有什么就标注什么
![20210318163348](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20210318163348.png)

2. 路面上的围栏标注为fence，路两侧建筑施工的围墙标注为building

3. 不确定的物体先不要标注

## 标注流程
1. 先用Remove with plane， 把地面以上的物体都过滤掉，然后把road标注出来
2. 同样的方法把sidewalk标注出来；
3. 把道路某一侧的绿化带过滤掉，然后标注路面上的围栏，对两侧的围栏都做这个操作
4. 用Remove with plane将路边的建筑围栏保留下来，并标注；
5. 用Remove with plane过滤掉路边围栏，并标注所有的树木和交通标识；
6. 标注车辆和行人，可以在用Remove with plane过滤到上方的树冠和建筑物之后，俯视图上标注，这时车辆、行人与地面接触的地方会有误标注，这部分先不需要处理；
7. 全部完成标注之后，处理行人、车辆、围栏与地面交界处的误标注label


