# PL-ICP
点对线的ICP

对于laser来说，由于激光是离散的，但是实际障碍物是连续的，如果是一个长的连续光滑障碍物，在其上的同一个点有可能在下一帧匹配到这个障碍物线段上任意一个点，匹配算法就会带来额外的误差。

因此要引入点到线的icp，如下图，最小化的其实是点到直线的距离，即最小化点在直线的法线方向的投影长度。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/08/15/1597496549705-1597496549717.png)

算法流程如下

