# VIO简介
imu和视觉各自的特性
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/12/27/1609078133926-1609078133962.png)

这里注意，VIO还是一种odometry，实际应用的时候需要确定到底需要相对运动估计(odometry)，还是需要绝对定位

VIO(或者是odometry)的应用可能为：手机拍照防抖，无人机追踪人，AR/VR

一般使用紧耦合的方式

注意这里的表示方式的顺序，Twi代表i系到w系的变换，即Twi右乘一个i系下的坐标得到其在w下的坐标
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/12/27/1609079525458-1609079525462.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/12/27/1609079802034-1609079802039.png)

四元数对时间求导：
首先是四元数求极限
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/12/27/1609080238018-1609080238021.png)
然后对时间求导
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/12/27/1609080407641-1609080407643.png)

常见的雅克比
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/02/1609521193207-1609521193239.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/02/1609521325683-1609521325684.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/02/1609521988768-1609521988772.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2021/01/02/1609522091236-1609522091238.png)

![20210306151707](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20210306151707.png)