# IMU运动模型

![20210306233717](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20210306233717.png)

![20210308002350](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20210308002350.png)


![20210308002009](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20210308002009.png)

# IMU确定性误差
![20220224163823](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224163823.png)

![20220224164021](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224164021.png)

![20220224164155](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224164155.png)
Run to Run: 每次上电之后的bias不一致
In run: 一次上电后, bias有一个缓慢的变化

## IMU确定性误差标定
![20220224164424](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224164424.png) 

![20220224164614](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224164614.png)

![20220224164836](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224164836.png)

![20220224164955](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224164955.png)

# IMU随机误差

## 高斯白噪声

![20220224165154](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224165154.png)

注意, 真实的IMU采样是离散过程, 而通常我们说的标准差$\sigma$是连续的标准差(通常和方差混淆说), 需要用一个公式将其转换为离散形式, 公式为

$$\sigma_d = \frac{\sigma}{\sqrt{\Delta t}}$$
其中$\sigma$是连续的标准差, $\Delta t$是采样时间(通常为1/传感器频率?)

![20220224165546](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224165546.png)

![20220224165929](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224165929.png)

## bias随机游走
![20220224170200](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224170200.png)
注意这里b的导数是高斯噪声, b本身是维纳过程.

![20220224170433](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224170433.png)

bias的噪声也存在连续形式到离散形式的变换, 变换公式为
$$\sigma_{b_d} = {\sigma_b}{\sqrt{\Delta t}}$$

为什么高斯噪声和bias游走的离散形式一个是除$\sqrt{\Delta t}$, 一个是乘呢? 一个直观的理解, 对于传感器测量的高斯噪声,采样越长, 噪声之间可能相互抵消了, 这时候噪声实际变小了, 体现出来就是时间越长方差越小, 是除法; 而bias的噪声, 是bias对时间t导数的方差, 时间越长, 求导越不准确, 也就是说时间越长, bias可能向未知的方向飘的越严重,表现为时间越短方差越小;


## IMU随机误差标定
![20220224171351](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224171351.png)

![20220224171822](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224171822.png)
在曲线上, 取t=1时对应的纵轴值作为高斯噪声的方差;
取t=3时对应的纵轴值作为bias的方差

![20220224172232](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224172232.png)

IMU标定工具: kalibr, imu_utils

注意allan标出来的不一定非常准确, 实际使用的时候基本要把方差放大10~20倍, 再放到系统利用, 认为imu的方差要比测出来的的大一点


# IMU数学模型
注意加速度计本身测量出来的是ENU系下的力, 需要转换到body系下
![20220224172941](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224172941.png)

![20220224173256](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224173256.png)


# VIO运动模型

![20220224184000](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224184000.png)

![20220224184206](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224184206.png)

## 运动模型的离散积分 
![20220224183804](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224183804.png)

![20220224183911](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224183911.png)


# IMU数据仿真
![20220224185406](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224185406.png)
其中思路2可以适用于某些深度学习数据集, 只有GT pose, 可以通过曲线拟合产生轨迹, 然后用轨迹插值采样点, 再求导得到imu测量

![20220224185706](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224185706.png)

## 如何用欧拉角将惯性系下的点旋转到body系?
![20220224190002](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224190002.png)

![20220224190118](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224190118.png)


## 如何将惯性系下欧拉角速度旋转到body系?

![20220224192648](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220224192648.png)

这里注意, 按照欧拉角惯性-body转换过程可知, 因为欧拉角的roll角速度就是绕着x轴旋转, 所以绕x轴的角速度是不用变换的, 在绕zy旋转之后,就是最终body系下的角速度了; 而原来pitch角速度是绕y轴的旋转, 还要再经过x轴的一次旋转, 所以要左乘roll角; 同理yaw角速度还要绕yx旋转