## 差分里程计模型
## 航迹推算
## 里程计标定

### 最小二乘方法
一个比较通用的算法是线性最小二乘方法。

基本上所有工程问题，都可以化为求解AX=b的优化问题。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/07/26/1595774233997-1595774234001.png)

这里注意，最小二乘的通解中，$A^TA$的条件数一般都很大，就会导致病态解，即b变化很小的时候，得到的解会有非常大的扰动，这对于实际问题是不可行的(最简单的比如曲线拟合问题)。这时候一般会对$A^TA$做QR分解，减小其条件数。

从线性空间的角度，看最小二乘的求解是这样的：
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/07/26/1595775313584-1595775313587.png)

其中Ax代表A的列向量空间，即A的所有列任意缩放，之后再组合张成的空间。

### 里程计标定