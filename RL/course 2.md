Markov Process with Reward and action
1. Markov过程: 由(状态,状态转移概率方程)构成
![Screenshot from 20190525 174358.png](0)

2. reward: 每个状态有一个reward(即每一个状态的reward是广播到其入边上的权值)

3. return: 是第t步的全部折扣reward,其中gamma决定了对未来可能的reward考虑多少
![Screenshot from 20190525 174538.png](2)

4. value: 对于t时刻的state S,value是指该state的所有reuturn的期望,即return是以S开头的一个采样序列的加权衰减reward,而value是值以s开头的所有采样序列的reward的期望(例如最简单的,平均数).
![Screenshot from 20190525 175437.png](3)

