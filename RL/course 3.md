#  Planning by Dynamic Programming

注意,这节课讲解的是Planning问题,而不是RL问题.

## Policy Iteration

Policy Iteration 是使用**Bellman期望方程**来求解. 用迭代的方法来求解最优的value fn,以及最优的策略.
这里我们可以使用**synchronous backups**方法,即:

在每一个迭代轮次k中, 对于所有的state, 使用上一个轮次的value fn $v_k(s')$, 根据Bellman公式来更新这一个轮次的value fn $v_{k+1}(s)$ (其中$s'$是s的后继状态,就是s采取action后能转移到的状态), 也即:

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/28/1558975694364-1558975694365.png)

- 开始的时候可以将所有的$v_0(s)$设置为随机值,并随机选取一个policy  $\pi_0$,然后重复迭代以下过程, 直到收敛:

- 在第k轮时, 首先使用上述的synchronous backups bellman方程, 来评估policy $\pi$(上一轮迭代后得到的), 得到这一轮的value fn $v_k(s)$;

- 然后,使用1 step look ahead的方法, 根据这个新的$v_k(s)$来更新policy, 得到 $\pi'$(这一过程可见ppt中的small grid world例子);

- 重复这两步, 最终能够保证$v(s)$收敛到最优$v^*(s)$, 且$\pi(s)$收敛到最优$\pi^*(s)$, 如下图所示(证明在后面给出).
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/28/1558976217618-1558976217623.png)

## Value Iteration

value iteration是使用**Bellman最优方程**进行求解.
与policy iteration不同, value iteration没有显式的policy,在迭代过程中,中间的$v(s)$并不代表任何policy(而在policy iteration).

使用vlaue iteration时, 按照以下步骤:

- 开始的时候可以将所有的$v_0(s)$设置为随机值,并随机选取一个policy  $\pi_0$,然后重复迭代以下过程, 直到收敛:

- 在第k轮时, 直接使用synchronous backups **bellman optimal**方程(即最优方程),得到下一轮的value fn, 计算出$v_{k+1}(s)$.

- 重复这两步, 最终能够保证$v(s)$收敛到最优$v^*(s)$, 且这个最优$v^*(s)$就对应着最优policy $\pi(s)$.

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559059488985-1559059488991.png)

## policy Iteration 和 Value Iteration 的对比
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559059648492-1559059648499.png)

## DP的改进方法

1. In place DP

不存储旧的$v(s)$,  而是计算了一次$v(s)$就立刻更新, 即
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559060741000-1559060741001.png)

例如, 在small grid world问题中, 在同步更新算法中更新了某个格子s的$v(s)$,是将这个值存到新的backup中,当更新s的相邻格子时,需要用到s的值(look ahead, 参考bellman方程), 这个时候用的仍然是旧的backup中的$v(s)$值;而在in-place算法中,不区分新的和旧的backup,只有一个backup,计算了$v(s)$后直接覆盖原来的值,二而计算s的邻居格子时,也总是用的最新的$v(s)$值.
这会大大增加计算效率, 但是也会导致计算某个状态s的多个后继状态时,这些后继状态用到的前驱状态s的value是不同的(虽然本应该都是同一个时刻的s值), 但是对一些问题,这样是没有问题的;对于其他的问题,可能涉及到顺序的问题. 

2. Prioritised Sweeping

如1中所说, 使用了in place之后,顺序决定了能提高的计算效率, 如果总是优先更新比较重要的state, 那么收敛速度也就能相应的加快. 因此,Prioritised Sweeping就采用了这种思想, 依据state的重要程度排序, 优先in-place更新重要的state.
计算state重要程度的度量是Bellman error,即一个state的value在更新前后的差值,直观上来讲,一个state在某次迭代中被更新的值越大,说明其对寻找
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559061629982-1559061629983.png)
