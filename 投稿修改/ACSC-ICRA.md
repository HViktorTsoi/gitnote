# 最大的贡献是提高了之前的工作ILCC的性能，但是不知道这个提升是不是由于给出了更多的信息（比如更多棋盘格的信息，如反射率信息）。已知intensity的使用使我很难将其视为自动校准算法，因为intensity将基于激光雷达的激光功率，目标距离等而有所不同，因此可能需要提前手动确定。

这里是reviewer没读懂，或者cost function那里写的不清除，不需要人工指定intensity的值，其值的高低是用来决定点云属于哪一类模式

# 和ILCC相比缺少创新性，在refine和checker localization上做的很好，但是当拿到checker之后，流程几乎和ILCC一致，在方法部分没有显式的引用ILCC

在cost function上，棋盘格投影方式，有区别

# 在实验部分，提出的方法是target-based的，不知道为什么要和Pandey 2015的targetless的对比，如果换成target-based会更fair

修改版本中仅和target-based的进行对比

# 作者说非重复性是SSL的特性，但是实际上有SSL是重复性扫描的(比如invoz)，也有机械雷达是非重复扫描的，这里应该对分类更明确

待解决，

# 作者说SSL的轴向误差很大，但是缺使用了法向分割方法来定位棋盘格，并且work了，这个需要说明。

说明在法向分割的过程中，邻域选择的范围比较大，因此会得到更平滑的法线估计；

# Scaramuzza et al. [20] proposed a geometric model-based… 引用文献错误

#  1. The article is interesting except for issues in language and mathematical expressions. Please help the reader to better understand and appreciate the article by making suitable corrections. 

# 2. Mathematical equations are used in a very confusing manner. It is very difficult to understand the terms and the reason for the expression.

# 3. In Figure 5, the authors are trying to show the reflectivity distribution of the checkerboard and the black-white pattern and claim they have the same spatial distribution. However, it is hard to observe how to relate the distributions. Please give more clear explanations.

因为篇幅问题，公式描述的不清晰，对所有的公式增加描述

# Fig 3, part b: Is camera intrinsic matrix used for extracting corners?

在问题描述中说明，已知相机内参

# Correct the journal details of reference [18]. 


# Concerning the result in Table 2: Does the pre-processing bias the results in favor of the proposed algorithm? Is it possible to do this without preprocessing to ensure fairness.

在实验部分增加完整的abalation study

# Please separate the images in Figures 1, 6 etc., if they do not need to appear together. Otherwise it will confuse the reader.  

将图1和图6重新排版

# Also please check if it is correct to alternate between the use of terms reflectivity and reflectance.

# Eqn (1) terms are not defined. What is N_H, N, P_{xy}...and so forth? Also what is H()? In Eqn (2) also there is similar problem.

# Terms in Eqn. 8 should be suitably defined. Also correct \tilde{p_i^x}, tilde{p_i^y} as \tilde{p}_i^x, \tilde{p}_i^y.

# In Fig 5 please mention the label in the y axis.

# Does || || and | | both represent euclidean norm? Please use it in uniform fashion.

# Average re-projection error of 2.11 is quite good. Whatis the reason for such a superior performance in-spite of the uncertainties in the sensor?
这里应该在abalation实验中明确每个部分的贡献，然后说明每个方法在SSL上表现出的问题；

# Authors may consider citing the following article.
Ranjith Unnikrishnan, Martial Hebert; Fast Extrinsic
Calibration of a Laser Rangefinder to a Camera;
CMU-RI-TR-05-09; 2005.

# Minor aspects:
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

# Unlike previous work, the claimed contribution of this paper is to make the best of the non-repetitive pattern of SSL utilizing temporal integration and feature refinement, which however does not theoretically contribute much to the state of art.

# The implementation of the idea is too simple and still at the abstract level

# More importantly, no theoretical or experimental comparison is done to claim the benefits of using the 3D and 2D corner features instead of planes - which is easy to get from Lidar point cloud.

这里需要通过实验验证，用checker的内角点会比面特征更高效

# After integrating the continuous Laser scans in the time-domain. An iteratively searching method is performed to find the best-fit plane and filter the outliers leveraging the RANSAC based method. Unfortunately, the accuracy of the proposed method is not reported in the paper or compared with any existing well-known RANSAC based methods in the literature. 

这里应该通过仿真实验来验证checker plane拟合的准确性

# The formulation of the cost function of 3D corner estimation is assumed to be one of the keys. The checkerboard model is given with zero z values but the transformations (Eq.7and8) are formulated as 2D transformations. Please give more detailed derivations.

# In Figure 8(a), it shows that by increasing the number of target placement will decrease the reprojection error. We can see a small jump when the number of target placements is Around 25-26. Please give some explanations why this happens.
这里实际上是用inner corner的优势，在两个position时收敛的效果已经很好了，要加一句说明

# And Figure 8(b) is trying to show the influence of integrated count to the calibration performance. From the figure, increasing the integrated frame count from 0 to 50 will cause the reprojection error from 0 to 20. Please show a more clear figure and give better explanations about this. Also, please demonstrate if the vertical axis of the figure represents the average reprojection error.

对图8b的横轴需要修改，加一致的颜色

# Please be careful with the notation, for example, at the left corner of Page 5, the measurement P_c should be bolded.


# 重点修改
1. 对ILCC的流程进行修改，突出创新性；
2. 突出挑战性场景，此时仅有基于有目标的标定方法才能work；
3. 做abalation study,描述的时候说明各个模块的作用；
4. 说明不用planefeature的原因，想要充分利用intensity 
5. 用箱线图表示，预处理的过程对各个传感器的重投影误差的影响(0.5)
6. 加上公式描述
7. 检测到的checkerboard的可视化(0.5)
8. 积分过程中，加入噪点，和降噪算法对结果的影响(0.5)
9. ~~积分的量对棋盘格特征显著性的可视化(0.5)~~
10. segmentation之后各个part的score和可视化
11. 将segmentation阶段展开说
12. 将resample阶段展开说(PCA之后进行重采样)
13. 量化评估integration对平面方差的影响
14. 在其他雷达模型上的泛化性能
15. 平面方差随距离的分布
16. 高度方向的宽度分布示意图
17. 最佳摆放位置实验
18. 每个图的纵坐标含义补充
19. 已知的参数, 中间参数 For Calirity
20. 和其他方法的对比:  
    1.  resample
    2.  LiDAR中的breaking point
    3.  没有考虑Non-repetitive scanning的noise model

21. 注意NRE不加绝对值
22. 不同LiDAR Model的resample 密度测试
23. upholder removal移动到localization section中
24. NRE应该是$d/d_{mean}*pixel或者是$$d/d_{min}*pixel$
25. people can explore， high precision using such kind of non-repetitive scanning LiDAR
26. 关闭arxiv的论文



# Introduction
Solid-State LidAR更加适用于在开放场景下的工业级别AR应用，比如远程驾驶，或者无人搜救，因为其有比RGBD相机更远的深度感知距离和更精确的深度测量能力，并且其成本要远低于高端的LiDAR。但是另一方面，上述的场景需要对LiDAR和相机之间的精确标定。

基于solid-state lidar的噪声模型进行降噪和feature refinement

然后，标定目标被从场景中定位 

target-based 方法是视觉传感器中最为广泛应用的方法. Duda 

target-based的方法相比于其他方法, 具有更高的精度和鲁棒性,并且标定误差用于溯源, 适用于可控环境下的精确标定.本文提出的方法采用target-based的方式, 


新改进:
1. 头图 30分钟
2. ~~第四章分成多个小节 10分钟~~
3. ~~第四章可视化的图做更改 30分钟~~
4. ~~第四章加上ablation study 2小时~~
5. ~~第四章多个方法对比的实验,换方法 2小时~~
6. ~~第四章加上3D可视化的图(标定/未标定对比)~~
7. 第三章对cost function进行更详细的说明
8. ~~确定NRE的计算方式,NRE换成NRE的绝对值~~
9. ~~所有的||和|要统一形式~~
10. ~~Livox表格的型号和数据补全~~
11. ~~pattern图的单位改成米~~
12. ~~第四章棋盘格位置摆放和积分时间 图像更正~~
13. ~~第四章多传感器标定效果 换title,分析结果~~
14. 第三章算法里的notation
15. 重新检查中算法的notation
16. ~~Fig10的描述要和图像对应上~~
17. ~~在segmentation那里,说明由于还没有进行feature refinement, 我们使用较大的邻域半径(大概是checkerboard宽度的一半)来进行法线估计~~
18. ~~跑和其他方法进行比较的的实验结果~~
19. ~~zhou的方法在related works里引用~~
20. ~~算法流程两个版本要统一~~
21. introdution加入
22. 调整参考文献引用顺序
23. ~~Fig10和11要改成|NRE|~~
24. ~~调整图的排版布局~~
25. ~~后边几个图重新排位置~~