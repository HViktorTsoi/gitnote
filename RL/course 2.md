# Markov Reward Process(MRP)

## Markov Process with Reward and action
1. Markov过程: 由(状态,状态转移概率方程)构成
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558778944853-1558778944854.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558778984856-1558778984857.png)

2. reward: 每个状态有一个reward(即每一个状态的reward是广播到其入边上的权值),注意t时刻的immediate reward为R~t+1~,因为这个课程的符号系统里,采集action是在t时刻,而得到"采取的action"的奖励和观测值的时候就被设置为t+1时刻了.也就是说t+1时刻就是本状态的immediate reward,即无论下一步转移到什么地方,都会计算这个reward值;或者说,当前state的immediate reward值是在退出这个state时才开始计算的.

3. return: 是第t步的全部折扣reward,其中gamma决定了对未来可能的reward考虑多少(不算第t步的)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558779005329-1558779005333.png)

4. value: 对于t时刻的state S,value是指该state的所有reuturn的期望,即return是以S开头的一个采样序列的加权衰减reward,而value是值以s开头的所有采样序列的reward的期望(例如最简单的,平均数).
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558779023901-1558779023903.png)

## Bellman 公式

根据$v(s)$的计算方式,有
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558796372023-1558796372025.png)

即$v(s)$可表示为递归形式,进一步的,可表示为
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558796439213-1558796439215.png)

则所有状态$s$的矩阵形式可表示为
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558796621415-1558796621416.png)

当已知$R$和$\gamma$和$P$时,即可求解$v(.)$

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/25/1558796855886-1558796855887.png)

# Markov Decision Process(MDP)

MDP在MRP的条件之下,增加了action,即在转移概率矩阵P和Reward中加入Action,
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/26/1558881185872-1558881185884.png)

1. Policy变为
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/26/1558881900232-1558881900232.png)

2. 此时状态转移矩阵变为所有action对应转移矩阵的期望,reward变为所有action对应的reward的期望,即
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/26/1558882025529-1558882025529.png)
注意,这时状态间的转移不再是随机过程,而是根据action去转移,这一点与MRP是不同的(在MRP中,某个状态$s$转移到$s'$是依据概率进行的,当然转移矩阵中的元素仍然是概率,只不过变为采取某个action的概率);

3. 某个状态$s$对应的state Value Function变为
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/26/1558882677961-1558882677961.png)
这个期望的求解方式为
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/26/1558883253036-1558883253037.png)
他告诉我们这个state有多好;

4. 在$s$下选择某个action的action value function为
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/26/1558882751185-1558882751187.png)
这个期望的求解方式为
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/26/1558883540784-1558883540785.png)
他告诉我们在某个状态$s$下,take某个action有多好(在$s$上take $a$之后能到达的所有下一个状态$s'$的value和)
