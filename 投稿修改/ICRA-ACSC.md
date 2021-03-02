## 最大的贡献是提高了之前的工作ILCC的性能，但是不知道这个提升是不是由于给出了更多的信息（比如更多棋盘格的信息，如反射率信息）。已知intensity的使用使我很难将其视为自动校准算法，因为intensity将基于激光雷达的激光功率，目标距离等而有所不同，因此可能需要提前手动确定。

这里是reviewer没读懂，或者cost function那里写的不清除，不需要人工指定intensity的值，其值的高低是用来决定点云属于哪一类模式

## 和ILCC相比缺少创新性，在refine和checker localization上做的很好，但是当拿到checker之后，流程几乎和ILCC一致，在方法部分没有显式的引用ILCC

在cost function上，棋盘格投影方式，有区别

## 在实验部分，提出的方法是target-based的，不知道为什么要和Pandey 2015的targetless的对比，如果换成target-based会更fair

修改版本中仅和target-based的进行对比

## 作者说非重复性是SSL的特性，但是实际上有SSL是重复性扫描的(比如)，也有机械雷达是非重复扫描的，这里应该对分类更明确

待解决，

## 作者说SSL的轴向误差很大，但是缺使用了法向分割方法来定位棋盘格，并且work了，这个需要说明。

说明在法向分割的过程中，邻域选择的范围比较大，因此会得到更平滑的法线估计；

## Scaramuzza et al. [20] proposed a geometric model-based… 引用文献错误

##  1. The article is interesting except for issues in language and mathematical expressions. Please help the reader to better understand and appreciate the article by making suitable corrections. 

## 2. Mathematical equations are used in a very confusing manner. It is very difficult to understand the terms and the reason for the expression.

## 3. In Figure 5, the authors are trying to show the reflectivity distribution of the checkerboard and the black-white pattern and claim they have the same spatial distribution. However, it is hard to observe how to relate the distributions. Please give more clear explanations.

因为篇幅问题，公式描述的不清晰，对所有的公式增加描述

## Fig 3, part b: Is camera intrinsic matrix used for extracting corners?

在问题描述中说明，已知相机内参

## Correct the journal details of reference [18]. 


## Concerning the result in Table 2: Does the pre-processing bias the results in favor of the proposed algorithm? Is it possible to do this without preprocessing to ensure fairness.

在实验部分增加完整的abalation study

## Please separate the images in Figures 1, 6 etc., if they do not need to appear together. Otherwise it will confuse the reader.  

将图1和图6重新排版

## Also please check if it is correct to alternate between the use of terms reflectivity and reflectance.

## Eqn (1) terms are not defined. What is N_H, N, P_{xy}...and so forth? Also what is H()? In Eqn (2) also there is similar problem.

## Terms in Eqn. 8 should be suitably defined. Also correct \tilde{p_i^x}, tilde{p_i^y} as \tilde{p}_i^x, \tilde{p}_i^y.

## In Fig 5 please mention the label in the y axis.

## Does || || and | | both represent euclidean norm? Please use it in uniform fashion.

## Average re-projection error of 2.11 is quite good. Whatis the reason for such a superior performance in-spite of the uncertainties in the sensor?
这里应该在abalation实验中明确每个部分的贡献，然后说明每个方法在SSL上表现出的问题；

## Authors may consider citing the following article.
Ranjith Unnikrishnan, Martial Hebert; Fast Extrinsic
Calibration of a Laser Rangefinder to a Camera;
CMU-RI-TR-05-09; 2005.

## Minor aspects:
1) Abstract: "...large quantity of studies..." may be
"...large number of studies..."
2) Page 1, column 1, para 1: "...the commonest one is..."
may be "...the most common one is..."
3) Page 1, column 1, para 2: "...However, The..." may be
"...However, the..."
4) Page 1, column 2, para 1: "...such drawbacks seriously
affected corresponding feature extraction in point clouds,
and thus" may be "...such drawbacks seriously affects the
feature extraction from point clouds..."
5) Page 1, column 2, para 2: "...and proposed a 3D corner
extraction method by utilizing the reflectance
intensity..." may be "...and propose a 3D corner extraction
method by utilizing the reflectance intensity..."
6) Page 1, column 2, para 2: "...Based on the 3D corners
and corresponding 2D corners(from the optical image), a
target-based calibration is proposed" may be reworded as "A
target based calibration that uses the extracted 3D corners
and 2D corners from the optical image."
7) Page 2, column 1, para 3: Please reword "The
targetless-based method tries"
8) Page 2, column 1, para 4:"Scaramuzza et al. [20]
proposed" is not correct. Please check and correct author
name.
9) Page 2, column 2, para 1: "...and achieve accuracy
calibration results.." may be "...and achieve accurate
calibration results.." 
10) Please state the unit of measurement in Fig 2.
11) Page 3, column 1, para 1: "..., the integrate the
noise-free points in time-domain." may be changed as a new
sentence ". The noise free points are then integrated."
Also consider about splitting the main sentence further to
aid in better clarity. 
12) Page 3, column 1, para 2: "that may contains the
checkerboard." may be reworded as "that may contain the
checkerboard."
13) Page 3, column 2, para 2: "due to due systematic" may
be corrected suitably.
14) Page 5, column 1, para 2: "...the RANSCA-based..." may
be corrected.
15) Page 5, column 1, para 2: "process continues
repeatedly, until all errors" may be reworded as "process
continues, until all errors"
16) Page 6, column 1, last par: "samples at 5^{\~}6 poses."
may be corrected.
17) There are numerous other language issues. Please check
in detail before final submission.
18) Format for Table captions may have to comply with IEEE
style. 

## Unlike previous work, the claimed contribution of this paper is to make the best of the non-repetitive pattern of SSL utilizing temporal integration and feature refinement, which however does not theoretically contribute much to the state of art.

## The implementation of the idea is too simple and still at the abstract level

## More importantly, no theoretical or experimental comparison is done to claim the benefits of using the 3D and 2D corner features instead of planes - which is easy to get from Lidar point cloud.

这里需要通过实验验证，用checker的内角点会比面特征更高效

## After integrating the continuous Laser scans in the time-domain. An iteratively searching method is performed to find the best-fit plane and filter the outliers leveraging the RANSAC based method. Unfortunately, the accuracy of the proposed method is not reported in the paper or compared with any existing well-known RANSAC based methods in the literature. 

这里应该通过仿真实验来验证checker plane拟合的准确性

## The formulation of the cost function of 3D corner estimation is assumed to be one of the keys. The checkerboard model is given with zero z values but the transformations (Eq.7and8) are formulated as 2D transformations. Please give more detailed derivations.

## In Figure 8(a), it shows that by increasing the number of target placement will decrease the reprojection error. We can see a small jump when the number of target placements is Around 25-26. Please give some explanations why this happens.
这里实际上是用inner corner的优势，在两个position时收敛的效果已经很好了，要加一句说明

## And Figure 8(b) is trying to show the influence of integrated count to the calibration performance. From the figure, increasing the integrated frame count from 0 to 50 will cause the reprojection error from 0 to 20. Please show a more clear figure and give better explanations about this. Also, please demonstrate if the vertical axis of the figure represents the average reprojection error.
