# Model-Free Control
## Monte-Carlo Control
Model-free MC控制其实和DP中的MC很相近,都是反复通过
采用策略,获得Q值->通过Q值采取贪婪地更新->采用策略,获得Q值
这样的迭代来获得最优策略和q值
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/06/11/1560267401367-1560267401448.png)

但是这里有个问题,由于在Model-free control中,无法得到系统的信息,即无法像DP那样得到所有的状态和状态之间转换的概率,因此如果一直采取贪心策略,可能卡在一种策略上,在局部最优不动了,这就是没有exploration的结果.

因此,为了增加exploration,提出$\epsilon$-greedy的exploration方法,其思路非常简单:
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/maste=r/gitnote/2019/06/11/1560268268987-1560268268988.png)
就是以$\epsilon$的概率随机选action,而以$(1-\epsilon)$的概率以贪婪策略选择action.同时种策略保证是收敛的,即保证采取$\epsilon$-greedy策略之后下一时刻的Q值是不小于当前时刻的.
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/06/11/1560268683346-1560268683379.png)

同时,不必等到采集到成千上万个样本序列,再更新Q值,而是可以每一个episode都进行一个小的更新:
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/06/11/1560268422113-1560268422114.png)

这里提出GLLE(Greedy in the Limit with Infinite Exploration)方法:
如果保证在迭代接近无穷轮时,所有state-action都被探索到,且越来越倾向于只使用贪婪策略($\epsilon$在后期趋近于0)这两点,就可称之为GLIE方法.
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/06/11/1560268751386-1560268751387.png)