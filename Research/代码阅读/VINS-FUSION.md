## 数据结构
### 1. FeatureManager
 管理特征, 包含window内的所有feature points

![20220217153349](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220217153349.png)

![20220217153133](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220217153133.png)

![20220217152325](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220217152325.png)

### 2. FeaturePerId
存储的是每一个路标点的信息. 就特征点P1来说，它被两个帧观测到，第一次观测到P1的帧为frame1,即start_frame=1，最后一次观测到P1的帧为frame2,即endframe()=2,并把start_frame~endframe() 对应帧的属性存储起来，对应的示意图如下：

![20220217152351](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220217152351.png)

```C++
class FeaturePerId//管理一个特征点
{
  public:
    const int feature_id;//特征点id
    int start_frame;//第一次出现该特征点的帧号
    vector<FeaturePerFrame> feature_per_frame;//管理对应帧的属性
    int used_num;//出现的次数
    double estimated_depth;//逆深度
    int solve_flag; // 0 haven't solve yet; 1 solve succ; 2 solve fail;该特征点的状态，是否被三角化
    Vector3d gt_p;
    FeaturePerId(int _feature_id, int _start_frame)//以feature_id为索引，并保存了出现该角点的第一帧的id
        : feature_id(_feature_id), start_frame(_start_frame),
          used_num(0), estimated_depth(-1.0), solve_flag(0)
    {
    }
    int endFrame();//得到该特征点最后一次跟踪到的帧号
};
```

### 3. FeaturePerFrame
每个路标点在不同的, 能观测到他的关键帧图像上,都会有不同的投影/速度信息, 这个结构是存储这样的信息. 在上面提到对应帧的属性这个概念，它指的是空间特征点P1映射到frame1或frame2上对应的图像坐标、特征点的跟踪速度、空间坐标等属性都封装到类FeaturePerFrame中，对应的示意图如下：
![20220217152629](https://cdn.jsdelivr.net/gh/HViktorTsoi/gitnote-image@master/PicGo/20220217152629.png)

```C++
class FeaturePerFrame//一个特征点的属性
{
  public:
    FeaturePerFrame(const Eigen::Matrix<double, 7, 1> &_point, double td)
    {
        point.x() = _point(0);
        point.y() = _point(1);
        point.z() = _point(2);
        uv.x() = _point(3);
        uv.y() = _point(4);
        velocity.x() = _point(5); 
        velocity.y() = _point(6); 
        cur_td = td;
    }
    double cur_td;//imu-camera的不同步时的相差时间
    Vector3d point;//特征点空间坐标
    Vector2d uv;//特征点映射到该帧上的图像坐标
    Vector2d velocity;//特征点的跟踪速度
    double z;
    bool is_used;
    double parallax;
    MatrixXd A;
    VectorXd b;
    double dep_gradient;
};
```

# Global Fusion

1. 输入的原始VIO转换到GPS对应的世界坐标系;
2. 找到与某一帧VIO同步的GPS坐标;
3. 每一帧VIO对应的全局位姿的R和T为优化变量, 共有两种约束来构成优化的cost
   1. 前端相邻两帧之间计算出来的相对VIO, 与对应相邻两帧优化变量的相对位姿 之差
   2. 某一帧对应的GPS全局坐标, 对应帧优化变量的位姿 之差