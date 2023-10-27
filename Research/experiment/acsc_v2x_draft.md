
\section{Motivation Study}
\label{sec:motivation}

\textbf{Conventional SLAM Approaches.} The basic idea of visual SLAM front-end is to track the associated 3D feature points from consecutive frames and use the triangulation method to recover the 3D positions of the points, and then estimate the ego-poses of the camera through multi-view geometry relationships. By accumulating the estimated poses and 3D points, the localization of the camera relative to the local frame can be obtained, as well as the environment map. However, because (1) the feature descriptors of the landmarks consistently change with the camera perspective, the feature matching and tracking process are not stable; (2) since SLAM is based on the static environment assumption, the features in the non-static area will have inconsistent motion with the static area, which also affects the accuracy of relative pose estimation; and (3) in actual deployment, the intrinsic and extrinsic sensors calibration error decrease the measurement accuracy of feature points, especially for a low-cost sensor like cameras. These errors will inevitably accumulate during frame-by-frame pose estimation, thus making localization accuracy worse during long-term driving. Most existing methods utilize loop closure detection\cite{arshad2021role} to tackle this problem, which detects camera revisit event through image matching and constructs loop constraint, and then eliminates the error by pose graph optimization. However, this method has strict requirements on the vehicle trajectory, only when the current and the historical location of the vehicle have a large overlap can the loop closure be detected. In actual driving scenarios, such a loop constraint is hard to achieve, because the vehicle barely drives through a similar path repeatedly during a short period. Moreover, even though the vehicle drives back to the same road, the loop detection can still fail due to the limited FOV of the camera.

We set up a real-world road test to evaluate the performance of conventional visual SLAM algorithms. A RealSense D455 (90-degree FOV, with an IMU) camera is mounted on the front of an autonomous vehicle, and a high-precision RTK-GNSS system is used to generate ground truth trajectory. We use ORB-SLAM3 \cite{campos2021orb}, a state-of-the-art SLAM algorithm that has been widely used by both research and industry communities, to estimate the camera pose while the vehicle is driving along the road and fuse the reconstructed 3D landmarks to obtain a sparse feature point map, as shown in Fig. \ref{fig:motivation_study} (a), where the color of the map points represents driving time. It can be seen that, due to lacking global constraint, there is an obvious accumulated drift in the estimated pose and map by ORBSLAM3 compared with the ground truth trajectory. At point A, the reconstructed ground plane of the two roads ought to overlap with each other since the vehicle is driving on a 2D road, however, there is an obvious Z-axis map drift at around 3m; then the error accumulates continuously, and at Point B, the drift has reached around 12m at Z-axis and 8m at Y-axis. Besides, the scale of the map has also shifted, which can be seen from the left part of the map, the width of the reconstructed map is slightly larger than the ground truth trajectory. The loop closure detection fails at both A and B since the current and the historical orientations mismatch, i.e., the vehicle's orientations while repeatedly passing the road are completely opposite. Therefore, there are not enough overlapped features to detect loop closure.

\textbf{Benifits of Infrastructure-assisted localization.} We also evaluate other localization approaches that are widely applied to consumer-grade vehicles, as shown in Fig. \ref{fig:motivation_study} (b). The precision of the commonly use GPS localization method is around 5m, there are sudden changes due to the multipath effect in the urban environment, and it is not available occasionally in the occluded environment such as overpasses(the discontinuous values in the GPS curve in Fig. \ref{fig:motivation_study} (c)). The performance of GPS localization can be increased by integrating IMU measurement, but it is only applicable to the situation where GPS is missing for a short period. The VSLAM-GPS fusion \cite{qin2019general} method utilizes GPS measurement as a global constraint to eliminate the accumulated error of the SLAM method, however, due to the large error of GPS itself, it can not provide a global constraint with high confidence in the fusion algorithm and the average precision is around 2m. Besides, under a GPS-denied environment such as a long tunnel and underground parking lot, the error of slam will continue to accumulate.

On the other hand, it can be seen from Fig. \ref{fig:motivation_study} (a) that the ground truth trajectory aligns well with the the road-side infrastructure, whose location has been pre-calibrated during the installation. The LiDAR mounted on the infrastructure can obtain an accurate 3D measurement of the surrounding environment. Taking the Livox Horizon as an example, its ranging accuracy is about 2cm within a range of 100m. If this measurement can be matched with the 3D map reconstructed by the vehicle, the precise location constraints can be obtained and utilized to optimize the SLAM map. Since the observation of infrastructure is real-time, the map can be updated with the latest environmental information every time the vehicle passes the infrastructure to maintain an up-to-date and global-constraint map. And the vehicle can also perform re-localization on this map to estimate precise relative pose in the infrastructure frame and thus improve the perception range of the vehicle.   





\section{System Design}
\label{sec:system}

% **** 4.1 加overview, 把keyidea写清楚

\subsection{System Overview}

The design objective of VILM is to achieve high precision visual mapping and localization with the aid of the 3D measurement from the roadside infrastructure. As shown in Fig. \ref{}, we first utilize the visual odometry and submap reconstruction to recover the 3D geometric information of the road environment, and also to obtain a accumulive estimation of the vehicle pose. While driving through the roadside infracture, the vehicle receives the static scene point cloud extracted from the infracture. Then, with the proposed elastic registration approach, the localization of the visual submap with respect to the infracture frame is estimated. We then design a factor graph-based method to fuse the registration resualt with the visual odmmetry, and a global consistent 3D map can be obtained after the graph optimization.


\subsection{Submap Construction}
Associating the 2D image to a 3D point cloud is challenging. In this work, based on the motion prior of the vehicle camera, we reconstruct the road environment with keyframe-based sparse visual odometry, to obtain a 3D submap with homogeneous geometric information as the infrastructure point cloud for the 2D-3D association. We first extract FAST features from each image and track them between every two frames via KLT. Then, the frame-to-frame relative pose is estimated and the initial 3D coordinate of the feature points is recovered by triangulation. Due to the scale ambiguity of visual reconstruction, the imu measurement is also used to recover the initial scale of the submap. The 3D feature points(named landmarks) and their corresponding camera pose as well as the imu measurement are accumulated into a sliding window, and then the submap is optimized through then visual-inertial bundle adjustment. The structure of the submap can be described by the combination of three sets 
\begin{equation}
    M = \{ \mathcal{T}, \mathcal{L}, \mathcal{O} \}
\end{equation}
where 
\begin{equation}
    \mathcal{T}=\{ T_i|T_i \in SE(3), i=1,...,n \}  
\end{equation}
is the poses of each keyframes, and
\begin{equation}
    \mathcal{L}={L_i|L_i \in R^3, i=1,...,m}    
\end{equation}
is the coordinate of the estimate landmarks, and 
\begin{equation}
    \mathcal{O}={<T_i, L_j, u, v>|T_i \in \mathcal{T}, L_j \in \mathcal{L}, u,v \in \mathbb{R}}  
\end{equation}
is the association between the poses and the landmarks. $<T_i, L_j>$ denotes the landmark $j$ is visible at camera pose $T_i$, and the observed pixel coordinate is $(u,v)$. The reconstruction is continuous during the vehicle driving, therefore the last pose $T_n$ can be an initial estimation of the current vehicle location, and the reconstructed landmarks $\mathcal{L}$ are kept for the subsequent submap registration and map optimization. 


\subsection{Vehicle-Infrastructure Alignment}

There are two main challenges to align the point cloud with the submap. First, the submap are integrated by accumulate the visual odometry with the estimated 3D landmarks from 2D images, which means that the error of the pose, scale and feature depth are also accumulated into the submap and causing the skew and scale-inconsistency of the map.  Therefore, the conventional rigid-based point cloud registration algorithm is hard to be applied. An intuitive case is that, after tigid registration, part region of the submap could match, while displacement exists in the remaining part. Second, the visual submap is sparse and hard to extract effective geometric feature. By utilizing the non-rigid assumption, we proposed a elastic registration approach which jointly associate the non-rigid map segments with the infracture cloud and optimize the landmarks coordinate to tackle these problems, as described below.


\subsubsection{Static Scene Extraction}
The point clouds obtained from the infrastructure LiDAR may be highly dynamic, especially for the scenario with heavy traffic flow. These dynamic points, such as vehicles and pedestrians, will cause the inconsistency between the visual submap and infrastructure point clouds. A straightforward idea to eliminate the influence is to classify the point cloud with semantic segmentation and then remove the points belonging to the possibly dynamic semantic category. However, inspired by \cite{kim2020remove}, we observe that, based on the prior that the infrastructure LiDAR is mounted on a fixed position with a known pose, the dynamic points can actually be filtered out by the historical spatial occupancy information. Therefore, We divide the space that can be observed by the infrastructure into multiple voxels and then accumulate the point clouds during a period of time. The voxels are then voted if they are occupied by any points, and finally, those voxels with low occupancy can be classified as dynamic regions. The raw points that belong to those static voxels are subsampled and then transmitted to the vehicle. The details are shown in Algorithm \ref{alg:static_scene_extraction}. And for the dynamic points in vehicle-side images, we remove the estimated 3D points with large reprojection errors before integrating them into the visual submap, since such points usually have an inconsistent motion with the static environment \cite{geneva2020openvins}. The benefit of this design is that the succeeding submap registration procedure can obtain a time-invariant reference point cloud, and it improves the registration robustness. Besides, by doing this, there is also no need for the infrastructure to transmit the point cloud to the vehicle in real-time, since the point cloud is a static measurement of the environment and is identical during a period of time, and in the scenario where an infrastructure serves multiple vehicles, the same point cloud can be broadcasted to them, which greatly improves the scalability of the system.


\begin{algorithm}
\caption{Static scene extraction}\label{alg:static_scene_extraction}
\KwData{A point cloud sequence $\mathcal{P}=\{P_0,P_1,...,P_n\}$ from the infrastructure LiDAR, Voxel size $V_g$, Occupancy threshold ${t}$.}
\KwResult{The point cloud $\mathbf{P_s}$ of the static scene.}
Initialize a 3D hash table $H(x,y,z) = 0$ \;
Initialize a index table $T(x,y,z) = \{\}$ \;
\For{each point cloud $P_i$ in $\mathcal{P}$}{
    
    Subsample $P_i$ \;
    
    \For{each point $p_j$ in $P_i$}{
    
        $K^x, K^y, K^z = \lfloor \frac{p^x_j}{V_g} \rfloor, \lfloor \frac{p^y_j}{V_g} \rfloor,\lfloor \frac{p^z_j}{V_g} \rfloor$ \;
        
        \If{$(K^x, K^y, K^z) \not\in H(x,y,z)$}{
            Initialize $H(K^x, K^y, K^z)$ \;
        }
        $H(K^x, K^y, K^z)++$ \;
        
        Append $p_j$ to $T(K^x, K^y, K^z)$ \;
    }
}
\For{each hash key $K^x, K^y, K^z$ in $H(x,y,z)$}{
    \If{$H(K^x, K^y, K^z) > t$}{
        Merge $T(K^x, K^y, K^z)$ into $P_s$ \;
    }
}
Downsample  $P_s$ \;
\end{algorithm}

% \begin{algorithm}
% \caption{Static scene extraction from the infrastructure point clouds.} 
% \label{alg:static_extraction} 
% \begin{algorithmic}[1] %这个1 表示每一行都显示数字
% \REQUIRE ~~\\ %算法的输入参数：Input
% The set of positive samples for current batch, $P_n$;\\
% The set of unlabelled samples for current batch, $U_n$;\\
% Ensemble of classifiers on former batches, $E_{n-1}$;
% \ENSURE ~~\\ %算法的输出：Output
% Ensemble of classifiers on the current batch, $E_n$;
% \STATE Extracting the set of reliable negative and/or positive samples $T_n$ from $U_n$ with help of $P_n$; 
% \label{ code:fram:extract }%对此行的标记，方便在文中引用算法的某个步骤
% \STATE Training ensemble of classifiers $E$ on $T_n \cup P_n$, with help of data in former batches; 
% \label{code:fram:trainbase}
% \STATE $E_n=E_{n-1}\cup E$; 
% \label{code:fram:add}
% \STATE Classifying samples in $U_n-T_n$ by $E_n$; 
% \label{code:fram:classify}
% \STATE Deleting some weak classifiers in $E_n$ so as to keep the capacity of $E_n$; 
% \label{code:fram:select}
% \RETURN $E_n$; %算法的返回值
% \end{algorithmic}
% \end{algorithm}

\subsubsection{Initial Alignment}
When the vehicle drives near the infrastructure, we query the infrastructure localization in the pose set $T$ of the vehicle, extract the neighboring pose set $T_sub$ that is close to the infrastructure, and then transform the associate landmarks to the world frame to construct the visual submap $M$. Thanks to our factor-graph-based design, the initial localization $T_i \in \mathcal{T}$ can be firstly estimated by fusing the visual odometry with historical position constraints of infrastructure. For the special case where the vehicle hasn't been associated with any infrastructures, we can also estimate the initial pose by fusing the visual odometry with the GPS measurement or the TOA-range measurement from \textcolor{red}{the infrastructure-vehicle communication device}. 

Due to the sparsity of the visual submap, the registration is first done on a coarser granularity with the aid of ground-plane semantic constraints. We estimate the ground-plane pose $T_{sub}$ from the submap and $T_{ipc}$ from infrastructure point clouds respectively. Notice the $x$, $y$ and $yaw$ part of the poses is zero since these axes are not observable from a plane estimation. Then we can estimate the pose that has eliminated relative roll, pitch, and z offset between the submap and infrastructure points
\begin{equation}
    T'_i = T_i \Delta T^{r,p,z} = T_i (T^{r,p,z}_{sub})^{-1} T^{r,p,z}_{icp}, ~T_i \in \mathcal{T}    
\end{equation}
Taking $T'_i$ as an initial solution, we utilize the sparse NDT approach to estimate a rough relative alignment between the submap and infrastructure point cloud. For the sake of simplicity, we still use $T_i \in \mathcal{T}$ as the initial pose of the visual submap with respect to the infrastructure point cloud. Therefore, in the succeeding procedure, the feature association can be searched in the local region of the point cloud. It is more robust than the global matching methods since the visual submap is usually sparse and it is hard to extract distinctive geometry features. Meanwhile, it also improves computing efficiency because brute-force global matching is avoided.

+++++++++++++++++++++++++++++
With this initial alignment, the final feature association is more robust than the global matching methods since the sparse submap is usually lack of distinctive geometry features. Moreover, avoiding the brute-force searching in the global matching, such design is computational efficient.
+++++++++++++++++++++++++++++

\begin{figure}[!htb]
\begin{center}
% \fbox{\rule{0pt}{2in} \rule{0.9\linewidth}{0pt}}
   \includegraphics[trim={65 140 60 90},clip, width=1\linewidth]{4_System/VIML_VLBA.pdf}
\end{center}
%   \caption{ From sparse LiDAR measurements, our approach synthesis corresponding high resolution images. A) the depth and intensity projection from raw point clouds; B) Dense feature projection from the proposed DFP Transform module; C) Synthesised front-view image.}
\caption{The proposed elastic registration algorithm. }
\label{fig:dynamic_removal}
\end{figure}


\subsubsection{Elastic Registration}


Instead of treating the visual submap as a whole rigid point cloud, we regard it as a sequence of multiple 3D keyframe segments that may have displacement with each other. Each segment has an adjustable pose $\mathbf{T_i} \in \mathcal{T}$ and a corresponding 3D point set $\mathcal{F_i}$, representing the keyframe pose from initial alignment and the triangulated keyframe feature points attached with it. To associate the submap with the infrastructure point cloud, for each point in $\mathcal{F}_i$, we first search its neighboring point set from the infrastructure point cloud, if these points form a plane,  then we estimate the parameter $(mathbf{n},mathbf{q})$, where $mathbf{n}$ is the normal vector of the plane and $mathbf{n}$ is an arbitrary fixed point on the plane. Then we can register the point clouds by minimizing the overall multi-segment point-to-plane distances

\begin{equation}
    r_d(\mathcal{T}, \mathcal{F}) =\sum_{ T_i \in \mathcal{T}}{ \sum_{p_j \in \mathcal{F}_i}{| \mathbf{n_{i,j}^T} \cdot ( \mathbf{T_i} \mathbf{p_j} - \mathbf{q_{i,j})} ) | } }      
\end{equation}
where $p_j$ is the 3D feature coordinate on camera frame that is observed at pose $T_i$, and $(\mathbf{n_{i,j}},\mathbf{q_{i,j}})$ neighboring plane parameter of $p_j$ from the infrastructure point clouds.

Meanwhile, $\mathcal{T}$ and $\mathcal{F_T}$ are also constrained by the visual reconstruction, which means that the projected coordinate of a landmark $p_j$ on each image(those who can observe $p_j$) with pose $T_i$, should be consistent with the pixel coordinate $u$ of the feature points that are directly extracted from the images, therefore we can also construct a visual measurement residual

\begin{equation}
    r_{vis}(\mathcal{T}, \mathcal{F}) = \sum_{ T_m\in \mathcal{T}}{ \sum_{ T_i \in \mathcal{T}} { \sum_{p_j \in \mathcal{F}_i} { \sigma_m^{ij} \lVert \pi(\mathbf{K}, \mathbf{T_m^{-1}} \mathbf{T_i} \mathbf{p_j}) - u_m^{ij} \lVert } } }
\end{equation}
where $\sigma_m^{ij} \in \{0,1\}$ is a binary value denotes whether landmark $p_j$ can be observed at camera pose $T_m$, which is known from the feature matching during visual odometry; $\pi(\mathbf{K}, X)$ is a function that projects the 3D point $X$ to image pixel coordinate with $\mathbf{K}$ as the camera intrinsic parameter.

Notice that, since a landmark may be covisiable in multiple images, it may has several corresponding $p_j$ \textcolor{red}{form ()}, therefore
\begin{equation}
    \mathcal{L}\subset \{\mathcal{X}=\mathbf{T_i} \mathbf{p_j}|\mathbf{T_i} \in \mathcal{T}, \mathbf{p_j} \in \mathcal{F}_i \}
\end{equation}
Thus, rather than optimize all the 3D feature points from each frame, we only need to maintain a shared landmark set $\mathcal{L}$, so the residual can be simplified as

% \begin{equation}
\begin{gather}
r_{reg} = r_{d}(\mathcal{T}, \mathcal{F}) + r_{vis}(\mathcal{T}, \mathcal{F})
\\
=r_{d}(\mathcal{T}, \mathcal{L}) + r_{vis}(\mathcal{T}, \mathcal{L})
\\
= \sum_{x_i \in \mathcal{L}}{| \mathbf{n_{i}^T} \cdot ( x_i - \mathbf{q_{i})} ) | }
+
 \sum_{ T_i \in \mathcal{T}} { \sum_{x_j \in \mathcal{L}} { \sigma_i^j \lVert \pi(\mathbf{K}, \mathbf{T_i^{-1}} x_j) - u_i^j \lVert } } 
\end{gather}
% \end{equation}

Then we can estimate the poses of each segment in the infrastructure frame as well as the refined landmark coordinates by minimizing $r_{reg}$
\begin{equation}
    \mathcal{T}_{reg}^*, \mathcal{L}_{reg}^* = \arg \min_{\mathcal{T},\mathcal{L}} {r_{reg}}    
\end{equation}

The poses in $\mathcal{T_{reg}}^*$ are kept for the later map optimization as global constraints and the landmarks as well as are registered merged to the final map.

\subsection{Map Optimization}

\subsubsection{Factor Graph Construction}
To fuse the result from visual reconstruction and submap registration in real-time, we utilized a factor graph-based approach, as shown in Fig. {}. Each key-frame poses $T_i \in SE3$ to be solved are the node of the graph. The relative poses estimated by the visual odometry between adjacent frames are used as edge factors to connect each node from the graph. Meanwhile, the available GPS position $T_{gps} \in R^3$ that roughly estimated the vehicle position is also added to this factor graph as a constraint. Due to the large GPS positioning error, the covariance of this factor is set to a large value to against affecting the robustness of the initial positioning. 

After the submap registration, we use the optimized keyframe pose $\mathcal{T_{reg}}^*$ as prior factors and then add each factor to its corresponding node in the graph. Notice that, because the vehicle can not determine its world frame localization by visual odometry, we transform all of the visual odometry to the world frame after the first submap registration, then add a prior factor constraint at the first node and assign a large covariance to the translation and yaw angle part. In the subsequent optimization process,  the first pose is adjusted continuously to ensure the start point of the map is aligned with the world frame. 

% \textcolor{magenta}{

% }

\subsubsection{Global Map Refinement}
Once a certain amount of visual odometry or a submap registration constraint has been added to the factor graph, we periodically optimize the graph so that we find the Maximum A Posteriori (MAP) estimate for the keyframe poses. Then the poses from the graph node are extracted to transform the 3D landmarks of the corresponding keyframes to the world frame and fused as the final map. Moreover, the last graph node is the current estimation of vehicle localization, and also serves as the initial pose if the vehicle is driving through the next infrastructure. 

To maintain a global-consistent map and avoid map size increment during long-term mapping, the redundant feature points in the map should be eliminated. However, because the feature measurement varies when the camera observes the same position from different perspectives, the straight-forward voxelization downsample method may lead to the loss of multi-perspective feature information, which will cause tracking lost during re-localization. Therefore we design a map merging method based on local feature consistency, as shown in Algorithm~\ref{}. \textcolor{red}{Add brief description of the algorithm}.



As shown in Fig. \ref{} above, the vSLAM denotes the  test vehicle's trajectory estimated by visual-SLAM, we use ORB-SLAM3 \cite{}, a state-of-the-art SLAM framework that has been widely used by both research and industry communities, to estimate the camera pose while the vehicle is driving along a non-loop road. The Ground-Truth denote the groundtruth trajectory recorded by the on-vehicle RTK-GNSS device, which has a 2-cm localization accuracy. The Infrastructure denotes the positions of the infrastructure LiDARs deployed along the roads. The color mapped on the vSLAM trajectory denotes its error compared with groundtruth trajectory. It can be seen that, the trajectory error is acceptable during a short distance (the error is less than 2m during the first 100m test), however, the there is an obvious accumulated drift with the trajectory increases，which reaches over 10m at the end of the test. This is because the trajectory is estimation in a dead-reckoning pattern, the frame-by-frame error is also accumulated into the estimation, and there is no global constraint to eliminate such error. Most conventional SLAM algorithms utilize loop closure detection \cite{} 

This why SLAM is mostly used in small-scale indoor scenario, rather than the driving scenario.

due to lacking global constraint, 
   

% 说明有很多infrastructure覆盖不到的地方也需要定位


% 增加一个图，是首次定位的误差折线图，重定位折线图，纯visual odometry的折线图， 回环的折线图画在一起

% 为了加以区分，首次建图称为mapping，第二次建图称为localization

% 数据集分为固定安装数据集和非固定安装数据集， 非固定安装数据集是为了验证Infra的部署方式对结果的影响

% 配准 画CDF图

% 对传输延迟的鲁棒性，用条形图和时间序列来表示 写作的时候，说明thanks to the factor graph design，我们可以使用所有的历史信息，因此某一个infra的传输失序不会影响最后总的地图质量。 另外写作的时候要说明是怎么模拟失序的。 用收到失序点云时刻的ATE表明失序的影响，用总的ATE表明整个建图系统对失序的鲁棒性

% 可视化场景，增加一个隧道的地图，体现VIO的累计误差，GPS的跳变，和infrastructure-assisted可以work的很好的效果

% 高精地图方案并不是一个持久的解决办法，无法应对更新的场景。我们这个主要就是对更新鲁棒

% 可视化的部分，说明构成了一个动态的高精地图




1. initial 车的箭头， 初始位置和车进来的事件应该体现在system中
2. poses的命名需要再确定
3. recon map放大一点
4. 系统框图应该加上通信的箭头
5. fig4不同的loss要加以区分
6. contribution第二点要突出解决了什么问题（解决了图像非刚体），而不是突出做了什么
7. 路端都放在左侧，剩下的全在车端做
8. 名称可以统一一下， 都叫map
9. broadcasting pointcloud， like GPS，突出广播的作用
10. coarse alignment要说明大概做了culling的操作，使得配准的点云大概一致


Introduction的流，有两个思路
1. 高精地图很重要-》 建图成本高，更新慢-》 提出了提成本建图方案，利用分布式的infrastructure measurements
2. 定位和建图很重要-》传统SLAM存在累计误差-》通常用回环检测消除，但是
3. INfrastructure-》可以解决定位问题-》传统的定位有哪些问题（VLSM，无线）-》我们使用Infrastructure的point cloud

motivation
1. 最先写setup

一步一步联系，subsection要在开头写明白更详细的小步骤

% motivation加一句 除此之外，VILM还可以用点云的measurements来优化构建的地图，这个使用GPS这类单点定位的sensor是很难实现的

% VILM可以构建的地图是带有视觉特征的三维几何地图，车辆可以在这个地图上进行重定位，还可以合并多个车辆构建的地图（白天，晚上不同季节的视觉地图都可以合并，可以保存某个位置上不同时间段的视觉地图，类似于历史playback的效果）

% multi-scale elastic registration， 用不同的submap长度来进行配准， 放在coarse alignment里进行配准

% 首先按照每一帧的方法来写，然后加一步消解，合并冗余的feature points

% 传统的non-rigid registration只管将三维点贴到ref上，

% introduction说明infrastructure的potision是可以pre-calibrate出来的，用RTK-GNSS-Total Station， 这个任何一个测绘局或者施工队都有，这个静态测量是毫米极精度的

% 说明只使用了图像和imu的数据，因此任意的low-cost camera和imu的组合都可以用来运行VILM

% fp16

% 因为infrastructure获取点云是异步的

% realtime的图那里，添加一个分割线，说明infrastructure的处理可以是非实时的

% typycal application scenario of VSLAM is small scale indoor-localization

% 回环检测，只能修复被回环约束的局部区域，对于没有回环的位置，其误差仍然会被累计

% infrastructure处理中，是否要体现panner feature的抽取

% 说明GPS的测量可以是特别低频的

% 说明ORB-SLAM是realtime的


项目概况（限1000字以内，含标点）



无人驾驶技术的快速发展， 正在推动中国人工智能、互联网、汽车行业、交通产业的融合创新与深刻变革。杭州市发改委发展”十四五”规划中, 着重强调了推进无人驾驶技术示范应用项目的落地以及加快车路协同基础设施的建设,完善促进智能网联汽车创新发展产业链。目前, 已广泛采用的单车无人驾驶系统范式，包含”感知-决策-执行”三大部分,而 作为运动-环境感知的核心前提, 精准的自主定位以及驾驶地图构建, 是支撑无人驾驶车辆安全行驶的关键技术.但是由于开放道路驾驶环境的复杂特点, 基于单车传感器的传统定位方法已经难以满足车辆在多种极端场景中的可靠性和安全性， 利用分布式路侧基础设施， 以车路协同的方式实现车辆的融合感知以及可靠定位，已成为加速无人驾驶应用真正落地的迫切需求。
随着城市建设的紧凑化、密集化，基于车载GNSS的传统定位方法，受多径效应及星况分布影响, 难以实现高层建筑、立交设施等长时间遮挡情况下的准确位置解算. 对此, 在GNSS拒止环境下融合激光雷达点云、相机图像等车载传感器的观测数据,以帧间特征匹配进行相对位置解算的同步定位与建图方法成为近年来的研究热点, 但由于受单车传感器有限的感知范围和道路环境的高度动态性影响, 其特征关联的稳定性难以保证, 且缺少全局性约束, 定位结果存在不可避免的累积偏移, 难以在真实道路场景下实际应用.  在此情况下, 基于路侧基础设施的车路协同定位方法正在逐渐展现出其优势. 路侧部署的激光雷达及相机等传感器可实现超视距动态物体检测，与车端传感数据融合，构成了定位所需的稳定关联特征， 并可以辅助构建完备的环境地图；同时由于路侧单元的静态特性, 可获取观测特征的单点绝对位置, 为车辆定位提供全局约束信息，通过联合优化的方式消除累计误差， 在复杂道路环境可以实现长行程的可靠定位。
本项目以构建复杂道路环境下高精度、低累计误差的无人驾驶协同位置感知方法为目标，聚焦智能网联汽车“环境感知技术”及“车联网云控平台”关键技术联合创新，通过融合分布式路侧传感器的观测数据，实现无人车辆的同步定位与环境地图构建；针对车-路双端相对定位的关键问题，设计观测数据时域稳定特征抽取方法，最小化路侧设施数据传输负载， 开展基于点云-图像及语义-几何多层级特征融合的传感器观测配准方法研究， 通过大规模因子图优化方法， 实现车辆行驶历史轨迹与分布式多路侧传感器相对定位的联合优化， 实时构建全局一致性环境地图。基于车路协同技术，构建更为精确可靠的无人驾驶定位系统， 符合杭州市及国家加快推进智能网联汽车产业的战略需求， 具有重要的理论和实践意义。




提高特征完备性, 提高特征匹配的鲁棒性

成为促使无人落地

研究


实现全局一致性环境地图的构建

从道路观测量抽取时间窗口内稳定静态时不变特征, 路侧单元只需要以广播形式传输单次特征即提供车-路双端观测量配准的全部信息, 无需维护大量的数据持续传输和连接开销






开展XXXX研究

分布式路侧(基础)设施协同的无人驾驶车辆(三维)同步定位与建图方法(研究)



在城市道路中


这里写一些无人驾驶
随着XXX复杂, 单车智能已经XXX, 



能够正在GPS拒止, 超视距, 道路环境高度动态, 恶劣天气


传统GPS定位的问题:道路环境复杂, 遮挡严重(把这里和单车智能的缺点结合在一起), 视觉定位的问题(累计误差, 受动态物影响), 半开放场景(狭窄区域), 造成安全隐患, 

另一方面, 由于道路环境的高度动态特性,车辆感知范围和FOV受限



解决传统视觉定位建图由于交通动态特性, 缺乏全局约束(更具体的词汇)和XXXXX(无人驾驶的challenge)导致的非线性, 非刚体累计误差, (这里需要扩写)  


基于单车的定位建图方式已经无法满足无人驾驶在极端情况下对于可靠性和安全性(扩充无人驾驶的要求)的要求.(全天候, 全场景)

多场景定位精度达到分米级; 


车路协同, 低成本高精度车辆定位, 并重建实时的高精度三维地图, 实现路侧设备与车辆的融合感知


在最小化路侧感知节点部署的情况下, 利用路侧单元的 实现车路两端的感知融合,消除动态物体影响(这里写一些关于SLAM动态物体导致优化出现误差的例子), 通过融合语义特征和几何特征, 实现XXXX刚体配准(类似BA和Elastic Fusion的内容)

最小化路侧计算成本和数据传输量


大规模因子图优化, 实现车端实时定位与路侧观测量约束的紧耦合, 实现不同的{时间长度粒度}, {空间范围粒度}下的紧耦合建图与定位算法,  




, 实现一致性建图; 同时实时结合建图信息, 实现车辆高精度(XX米级)定位(讲一下SLAM的simultaneously特性)



依赖现有路侧部署的传感器设备

从道路观测量抽取时间窗口内稳定静态时不变特征, 路侧单元只需要以广播形式传输单次特征即提供车-路双端观测量配准的全部信息, 无需维护大量的数据持续传输和连接开销

使其能够服务更多车辆

基于定位结果, 融合车路两侧感知信息, 构建局部驾驶语义地图, 

同时可利用多车维护本地地图, 利用分布式路侧XXXXXX, 将地图进行聚合(想更好的表达方式),   实现道路高精地图的实时更新

挑战 
1., 充分利用道路两侧的语义信息和几何特征, 准确实现车路双端感知数据的配准(对齐), 解决车端传感器的累计误差和高动态性, 这是实现高精度定位的关键和本质, 并能够应对低成本传感器FOV受限, 测量误差(传感器问题)等问题

2. 最小化路侧传感器与任意车辆的数据传输带宽与传输时延迟(路侧有LiDAR, camera等数据量极大), 以能够
3. 在最小化通信代价和计算复杂性的情况下, 能充分利用将车辆行驶轨迹过程中道路部署的分布式传感器提供的观测量,, 传感器的历史测量(也)融合到定位过程中, 作为时间-空间历史约束, 构建全局一致性的地图(这里想更好的表达)


定位的要求是点精确度, 车体在当前的位置对上就可以; 但是建图的要求, 却是空间精确度, 任意位置的任意地图特征/三维点/尺度信息都有很高的要求, 因为地图的下游任务可能会出现在(想更好的表达)地图的任意空间位置. 同时还有实时性的要求

做的是什么

实时构架环境的三维感知地图, 同时实现无人驾驶车辆的高精度定位



广播式, 大规模, 可扩展


# argument
初始解可以比较精确， 比如我们可以用静止GPS叠加的方式 获取到非常精确地单点定位（在室外停车或者交通灯附近），但是当车运动起来之后这个方法就不可以了；还可以用零速修正的方式


# Sensys2022 Indoor Smartphone SLAM with Learned Echoic Location Features
## Section Title & Paragraph start Phrasing
5 Design of ELF-SLAM
From the measurement study, the acoustic echoes exhibit sub-meter spatial distinctness, which is the basis of the fingerprint approach.

5.1 Approach Overview
        As illustrated in Fig. 7, the mapping phase of ELF-SLAM consists of two major components:
    Trajectory map construction:
        A user holds a smartphone and moves within the target indoor space to collect the acoustic echoes and IMU data simultaneously on the movement trajectory
    
    Trajectory map superimposition
        When multiple trajectory maps are available (e.g., through crowdsensing), they are su- perimposed to generate a floor map.

5.2 Graph-based SLAM Formulation
    Graph-based SLAM [26] constructs a graph whose nodes represent the mobile’s poses and edges represent the kinetic constraints relating two poses. 

5.3 ELF For Loop Closure Detection
        Identifying an effective feature for loop closure detection is critical to SLAM. In this section, we first demonstrate the ineffectiveness of the generic features. Then, we propose using CL to construct a learning-based feature.

    5.3.1 Ineffectiveness ofgeneric features.
        We conduct a controlled experiment to evaluate several generic acoustic features.
    
    5.3.2 Learning-based ELF.
            The ineffectiveness of generic features motivates us to apply CL to construct a custom feature, i.e., ELF. In what follows, we present ...
            
        Data pairing
            constructs positive/negative data pairs needed by CL. In image recognition tasks, the positive samples are constructed by introducing spatial perturbations such as resizing, cropping, and blurring, which do not erase the information needed for image recognition. However,
        
        Model pre-training
            exploits self-supervised learning to build a basic ELF extractor, which will be specialized by the model fine- tuning step presented later.

        Model fine-tuning
            uses a small amount of unlabeled data collected by users in a specific target space to adapt the pre-trained model to capture the environment-specific characteristics.

    5.3.3 Loop closure detection using ELF. 
        The last row of Fig. 8 shows ?? + 58 × 2, and ?? + 58 × 3 as marked by the green arrows. Fig. 11 the ESS trace computed using ELF. It shows peaks at footstep ?? + 58,

5.4 Loop Closure Curation
    5.4.1 Approach design
            We propose a clustering-based loop closure curation approach that is based on an ESS matrix defined as follows

        ESS matrix
            Consider a user’s trajectory consisting of ?? foot-1) × (??−1) ESS matrix, where the (??, ??)th element is the ELF-based steps. The pair-wise ESSs between any two footsteps form a (?? −

        Clustering-based approach for loop closure curation
            The goal of loop closure curation is to remove the false positives from the binarized ESS matrix. Due

    5.4.2 Effectiveness ofloop closure curation
        To demonstrate the impact of the false positives on SLAM, we use all positives in Fig. 12 as the loop closure information to construct the trajectory map. The

5.5 Trajectory Map Superimposition
    The trajectory map constructed from a single user’s data only contains the echo data on a specific trajectory. For real applications, it is desirable to combine many trajectory maps to form a floor map that covers most/all accessible locations.

5.6 Localization
    Once a map (either a trajectory map or floor map) is constructed, a smartphone’s location can be determined after capturing the echoes in response to the chirps. We consider two localization approaches...
