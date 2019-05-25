Markov Process with Reward and action
1. Markov过程: 由(状态,状态转移概率方程)构成
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558778944853-1558778944854.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558778984856-1558778984857.png)

2. reward: 每个状态有一个reward(即每一个状态的reward是广播到其入边上的权值),注意t时刻的immediate reward为R~t+1~,因为这个课程的符号系统里,采集action是在t时刻,而得到"采取的action"的奖励和观测值的时候就被设置为t+1时刻了.也就是说t+1时刻就是本状态的immediate reward,即无论下一步转移到什么地方,都会计算这个reward值;或者说,当前state的immediate reward值是在退出这个state时才开始计算的.

3. return: 是第t步的全部折扣reward,其中gamma决定了对未来可能的reward考虑多少(不算第t步的)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558779005329-1558779005333.png)

4. value: 对于t时刻的state S,value是指该state的所有reuturn的期望,即return是以S开头的一个采样序列的加权衰减reward,而value是值以s开头的所有采样序列的reward的期望(例如最简单的,平均数).
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558779023901-1558779023903.png)

