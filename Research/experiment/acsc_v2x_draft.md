


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
