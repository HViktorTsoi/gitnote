# laser分类
1. 三角测距
2. TOF

## TOF
由于光速很快，想求得返回时间需要非常非常高的晶震频率，这样成本会很高，所以一般是求相位差，发射一个连续波，接受一个连续波，比较他们的相位差，即作为时间差

# 激光雷达数学模型

**AMCL里包括似然场模型和光束模型**

## 光束模型

激光雷达可以用光束模型来描述，光打到物体上的每一种情况，都可以用一种概率分布来建模，那么在一个地图中，给定了位置之后所有点的上述概率的联合分布，就是这个位置在地图中可能的概率。
 ![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597477428164-1597477428200.png)

但是光束模型是有缺点的：
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597478039975-1597478039977.png)  

## 似然场模型
将有障碍物的鸟瞰地图进行高斯模糊，那么模糊之后的图就是似然场，数值的高低就代表此处有障碍物的概率，这样就不用计算概率了，对于一帧激光，只需要到似然场里查一下表(获取似然场score图像对应的像素)，就能得到此激光对应的障碍物概率。

这一方法计算量低，适用于结构化和非结构化环境，并且似然场是平滑的，不会因为位姿的噪声导致概率的突变。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597478928138-1597478928149.png)

似然场的缺点也是平滑，比如前面有一堵墙，在似然场中，无论laser打到墙前还是墙后还是墙上，得分都比较高，因此似然场就会弱化环境中的几何约束。(但是这种情况在现实中不严重)

# laser运动畸变
产生原因：
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597479613054-1597479613056.png)

矫正方法：
## 纯估计方法


### 理想ICP
使用ICP。这么看ICP的方法其实和互相关信号同步比较类似，将a b两帧点云都挪到原点之后，求解一个旋转矩阵R，使得a和Rb之间的协方差最小。这里的方法就用到了SVD。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597480093803-1597480093805.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597480124046-1597480124047.png)

但是上述的方法需要知道对应点。如果不知道对应点(实际情况)，就需要进行迭代计算(EM算法, ICP方法是其的一个特例)。

### 迭代ICP
EM算法的思路是，我们需要求A，但是求A的前提条件是需要求出B，那么EM算法的流程就是：
1. 先固定A，求B；
2. 得到B之后，固定B，再求A；
3. 固定A。。。，如此迭代。
EM算法典型的有ICP，隐马尔科夫模型。

因此基于EM算法的迭代ICP流程如下：
A是R,t; B是点对应(在EM中的隐变量)。
1. 寻找对应点，在这里我们直接用每个点在另一帧点云的最近点作为对应点；
2. 根据对应点，求解R T；
3. 对点云进行转换，计算误差，然后求新位置的对应点；
4. 不断迭代，直到误差小于阈值。

### VICP
考虑机器人速度的ICP：
假定匀速运动，那么如果已知上一时刻的位姿，还有一帧激光中每一个线的到达时间，那么当然可以通过机器人速度(包括线和角速度)，算出来每一个线实际对应的位置，如下图：
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597480967048-1597480967059.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597481484878-1597481484890.png)

LOAM和VICP很像。

## 里程计辅助方法
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597481923983-1597481923986.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597482203913-1597482203915.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597482442453-1597482442464.png)

由于收到laser时的可能没有里程计与其对应，因此需要对里程计进行插值：将里程计视为匀速或者匀加速的，可以进行一次或者二次插值
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597483069499-1597483069501.png)