https://medium.com/@jonathan_hui/rl-natural-policy-gradient-actor-critic-using-kronecker-factored-trust-region-acktr-58f3798a4a93
trust region 优化: 是优化lower bound 函数的过程
自然梯度是从Trust region推导过来的
满足Trust region的策略梯度,推导出最终参数的梯度就是自然梯度



# 稀疏解
对稀疏解的一个直观理解是，在使得Loss最小的情况下， 尽可能的让更多的$W$的L1 Norm接近0，即获得一个稀疏的$W$。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/05/23/1590168850361-1590168850384.png)

# 点云法线
对邻居点做PCA之后， 特征值最小对应的那个基(在U中的最后一列)对应着原始点集方差最小的方向，也就是法线平面的反向，所有点基本都靠在这个平面上了。

# CMAKE pybind
CMAKE中 如果想要使用指定版本的python对应的pybind11，需要将对应版本python的inclue和lib都引入进来，如下例子所示，其中PYTHON_ROOT是python对应的environment的根目录，如~/anaconda/env/py36，PYTHON_VERSION是python的版本,如3.6
```
# python and pybind11
set(PYTHON_INCLUDE_DIRS "${PYTHON_ROOT}/include/;${PYTHON_ROOT}/include/python${PYTHON_VERSION}m")
set(PYTHON_LIBARIES "${PYTHON_ROOT}/lib/")

message("PYTHON INCLUDE DIRS: " ${PYTHON_INCLUDE_DIRS})
message("PYTHON LIBRARY: " ${PYTHON_LIBARIES})
include_directories(${PYTHON_INCLUDE_DIRS})

## compile mls upsample library
add_library(upsample_ext SHARED upsample.cpp upsample.hpp)
set_target_properties(upsample_ext PROPERTIES PREFIX "")
#target_link_libraries(upsample_ext pybind11::module ${PCL_LIBRARIES})
target_link_libraries(upsample_ext ${PYTHON_LIBARIES} ${PCL_LIBRARIES})
```